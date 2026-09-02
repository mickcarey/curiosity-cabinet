# Cutscene Implementation Patterns

## Technical Approaches to Cinematic Control

This document explores implementation strategies for cutscenes in games, focusing on the transition between gameplay and cinematic moments, with TypeScript examples.

## Core Technical Challenges

1. **State Transitions** - Smoothly switching between gameplay and cutscene modes
2. **Input Management** - Handling player control during different cutscene types
3. **Camera Control** - Managing camera transitions and cinematic angles
4. **Animation Synchronization** - Coordinating multiple animated elements
5. **Memory Management** - Loading/unloading cutscene-specific assets

## Anti-Pattern: State Flag Chaos

### Bad Example
```typescript
// ❌ Multiple flags lead to invalid states and confusion
class GameController {
  isInCutscene: boolean = false;
  isPlayerControllable: boolean = true;
  isCameraLocked: boolean = false;
  isUIHidden: boolean = false;
  isDialoguePlaying: boolean = false;
  canSkipCutscene: boolean = false;

  startCutscene(): void {
    this.isInCutscene = true;
    this.isPlayerControllable = false;
    this.isCameraLocked = true;
    this.isUIHidden = true;
    // Forgot to set canSkipCutscene!
    // What if dialogue starts during cutscene?
    // Can have isInCutscene=false but isCameraLocked=true
  }
}
```

## Pattern: Finite State Machine for Cutscene Control

### Good Example: Explicit State Management
```typescript
// ✅ Clear states with well-defined transitions
type GameState =
  | 'gameplay'
  | 'cutscene_starting'
  | 'cutscene_playing'
  | 'cutscene_interactive'
  | 'cutscene_ending'
  | 'paused';

interface StateConfig {
  playerControl: boolean;
  cameraControl: 'player' | 'cinematic' | 'hybrid';
  uiVisible: boolean;
  canPause: boolean;
  canSkip: boolean;
}

class GameStateMachine {
  private currentState: GameState = 'gameplay';
  private previousState: GameState = 'gameplay';

  private stateConfigs: Map<GameState, StateConfig> = new Map([
    ['gameplay', {
      playerControl: true,
      cameraControl: 'player',
      uiVisible: true,
      canPause: true,
      canSkip: false
    }],
    ['cutscene_playing', {
      playerControl: false,
      cameraControl: 'cinematic',
      uiVisible: false,
      canPause: true,
      canSkip: true
    }],
    ['cutscene_interactive', {
      playerControl: true,
      cameraControl: 'hybrid',
      uiVisible: false,
      canPause: false,
      canSkip: false
    }]
  ]);

  private transitions: Map<GameState, Set<GameState>> = new Map([
    ['gameplay', new Set(['cutscene_starting', 'paused'])],
    ['cutscene_starting', new Set(['cutscene_playing', 'cutscene_interactive'])],
    ['cutscene_playing', new Set(['cutscene_ending', 'gameplay'])],
    ['cutscene_interactive', new Set(['cutscene_ending', 'cutscene_playing'])],
    ['cutscene_ending', new Set(['gameplay'])]
  ]);

  transition(newState: GameState): boolean {
    const validTransitions = this.transitions.get(this.currentState);

    if (!validTransitions?.has(newState)) {
      console.error(`Invalid transition: ${this.currentState} -> ${newState}`);
      return false;
    }

    this.onExitState(this.currentState);
    this.previousState = this.currentState;
    this.currentState = newState;
    this.onEnterState(newState);

    return true;
  }

  private onEnterState(state: GameState): void {
    const config = this.stateConfigs.get(state)!;

    gameInput.setEnabled(config.playerControl);
    camera.setMode(config.cameraControl);
    ui.setVisible(config.uiVisible);

    switch (state) {
      case 'cutscene_starting':
        this.prepareForCutscene();
        break;
      case 'cutscene_ending':
        this.cleanupAfterCutscene();
        break;
    }
  }

  private onExitState(state: GameState): void {
    // Cleanup logic for leaving states
  }
}
```

## Pattern: Cutscene Timeline System

### Bad Example: Hardcoded Timing
```typescript
// ❌ Brittle, hard to modify, timing dependencies
class Cutscene {
  async play(): Promise<void> {
    camera.moveTo(position1);
    await wait(2000);

    character1.playAnimation('wave');
    await wait(500);

    dialogue.show('Hello there!');
    await wait(3000);

    character2.playAnimation('nod');
    await wait(1000);

    // If any timing changes, everything breaks
  }
}
```

### Good Example: Timeline-Based System
```typescript
// ✅ Flexible, maintainable, easy to modify
interface TimelineEvent {
  startTime: number;
  duration?: number;
  execute(): void | Promise<void>;
  cleanup?(): void;
}

class CutsceneTimeline {
  private events: TimelineEvent[] = [];
  private currentTime: number = 0;
  private activeEvents: Set<TimelineEvent> = new Set();
  private isPaused: boolean = false;

  add(startTime: number, event: Omit<TimelineEvent, 'startTime'>): this {
    this.events.push({ startTime, ...event });
    this.events.sort((a, b) => a.startTime - b.startTime);
    return this;
  }

  async play(): Promise<void> {
    this.currentTime = 0;
    const startTime = performance.now();

    return new Promise((resolve) => {
      const update = () => {
        if (!this.isPaused) {
          this.currentTime = performance.now() - startTime;
          this.processEvents();
        }

        if (this.isComplete()) {
          this.cleanup();
          resolve();
        } else {
          requestAnimationFrame(update);
        }
      };

      update();
    });
  }

  private processEvents(): void {
    for (const event of this.events) {
      const isActive = this.activeEvents.has(event);
      const shouldStart = this.currentTime >= event.startTime;
      const shouldEnd = event.duration &&
                       this.currentTime >= event.startTime + event.duration;

      if (!isActive && shouldStart && !shouldEnd) {
        event.execute();
        this.activeEvents.add(event);
      } else if (isActive && shouldEnd) {
        event.cleanup?.();
        this.activeEvents.delete(event);
      }
    }
  }
}

// Usage
class CutsceneBuilder {
  buildIntroScene(): CutsceneTimeline {
    return new CutsceneTimeline()
      .add(0, {
        execute: () => camera.startTransition(cinematicPosition, 2000)
      })
      .add(500, {
        execute: () => fadeOut.start(500)
      })
      .add(1000, {
        execute: () => character.playAnimation('enter_scene'),
        duration: 3000
      })
      .add(1500, {
        execute: () => dialogue.show('The adventure begins...'),
        duration: 2000,
        cleanup: () => dialogue.hide()
      })
      .add(4000, {
        execute: () => camera.startTransition(gameplayPosition, 1000)
      });
  }
}
```

## Pattern: Interactive Cutscene System

### Good Example: QTE and Choice Implementation
```typescript
// ✅ Flexible system for interactive moments
interface InteractionPrompt {
  type: 'button' | 'choice' | 'movement';
  timeLimit?: number;
  onSuccess?: () => void;
  onFailure?: () => void;
}

class InteractiveCutsceneController {
  private currentPrompt: ActivePrompt | null = null;

  async promptButton(
    button: string,
    timeLimit: number = 3000
  ): Promise<boolean> {
    return new Promise((resolve) => {
      const prompt = new ButtonPrompt(button, timeLimit);

      prompt.onComplete = (success: boolean) => {
        this.clearPrompt();
        resolve(success);
      };

      this.setPrompt(prompt);
    });
  }

  async promptChoice(
    choices: string[],
    timeLimit?: number
  ): Promise<number> {
    return new Promise((resolve) => {
      const prompt = new ChoicePrompt(choices, timeLimit);

      prompt.onComplete = (choiceIndex: number) => {
        this.clearPrompt();
        resolve(choiceIndex);
      };

      this.setPrompt(prompt);
    });
  }
}

class ButtonPrompt {
  private startTime: number;
  private inputHandler: (key: string) => void;
  public onComplete?: (success: boolean) => void;

  constructor(
    private button: string,
    private timeLimit: number
  ) {
    this.startTime = performance.now();
    this.setupInput();
    this.showUI();
  }

  private setupInput(): void {
    this.inputHandler = (key: string) => {
      if (key === this.button) {
        const reactionTime = performance.now() - this.startTime;
        this.complete(true, reactionTime);
      }
    };

    input.on('keypress', this.inputHandler);

    // Timeout
    setTimeout(() => {
      this.complete(false, this.timeLimit);
    }, this.timeLimit);
  }

  private complete(success: boolean, time: number): void {
    input.off('keypress', this.inputHandler);
    this.hideUI();
    this.onComplete?.(success);

    // Log for analytics
    analytics.track('qte_result', {
      success,
      reactionTime: time,
      button: this.button
    });
  }
}

// Branch cutscenes based on interactions
class BranchingCutscene {
  private branches: Map<string, CutsceneTimeline> = new Map();

  async play(): Promise<void> {
    const timeline = new CutsceneTimeline();

    // Initial sequence
    timeline.add(0, {
      execute: () => this.playIntro()
    });

    // Interactive moment
    timeline.add(3000, {
      execute: async () => {
        const controller = new InteractiveCutsceneController();
        const choice = await controller.promptChoice(
          ['Save the village', 'Chase the villain'],
          5000
        );

        if (choice === 0) {
          await this.playBranch('save_village');
        } else {
          await this.playBranch('chase_villain');
        }
      }
    });

    await timeline.play();
  }

  private async playBranch(branchName: string): Promise<void> {
    const branch = this.branches.get(branchName);
    if (branch) {
      await branch.play();
    }
  }
}
```

## Pattern: Camera Control Systems

### Good Example: Smooth Camera Transitions
```typescript
// ✅ Sophisticated camera management
interface CameraState {
  position: Vector3;
  rotation: Quaternion;
  fov: number;
}

class CinematicCameraController {
  private gameplayCamera: CameraState;
  private cinematicCamera: CameraState;
  private currentBlend: number = 0;
  private targetBlend: number = 0;
  private blendSpeed: number = 1;

  private cinematicTracks: Map<string, CameraTrack> = new Map();

  startCinematic(trackName: string, blendDuration: number = 1000): void {
    // Save current gameplay camera
    this.gameplayCamera = this.captureCurrentCamera();

    const track = this.cinematicTracks.get(trackName);
    if (!track) return;

    this.targetBlend = 1;
    this.blendSpeed = 1000 / blendDuration;

    track.play();
  }

  endCinematic(blendDuration: number = 1000): void {
    this.targetBlend = 0;
    this.blendSpeed = 1000 / blendDuration;
  }

  update(deltaTime: number): void {
    // Smooth blend
    const blendDelta = this.blendSpeed * (deltaTime / 1000);

    if (this.currentBlend < this.targetBlend) {
      this.currentBlend = Math.min(this.currentBlend + blendDelta, 1);
    } else if (this.currentBlend > this.targetBlend) {
      this.currentBlend = Math.max(this.currentBlend - blendDelta, 0);
    }

    // Apply blended camera state
    const blended = this.blendCameraStates(
      this.gameplayCamera,
      this.cinematicCamera,
      this.easeInOutCubic(this.currentBlend)
    );

    this.applyCamera(blended);
  }

  private easeInOutCubic(t: number): number {
    return t < 0.5
      ? 4 * t * t * t
      : 1 - Math.pow(-2 * t + 2, 3) / 2;
  }

  private blendCameraStates(
    a: CameraState,
    b: CameraState,
    t: number
  ): CameraState {
    return {
      position: Vector3.lerp(a.position, b.position, t),
      rotation: Quaternion.slerp(a.rotation, b.rotation, t),
      fov: lerp(a.fov, b.fov, t)
    };
  }
}

class CameraTrack {
  private keyframes: CameraKeyframe[] = [];

  addKeyframe(time: number, state: CameraState, easing: EasingFunction): void {
    this.keyframes.push({ time, state, easing });
    this.keyframes.sort((a, b) => a.time - b.time);
  }

  getStateAtTime(time: number): CameraState {
    // Find surrounding keyframes
    let prev = this.keyframes[0];
    let next = this.keyframes[this.keyframes.length - 1];

    for (let i = 0; i < this.keyframes.length - 1; i++) {
      if (this.keyframes[i].time <= time && this.keyframes[i + 1].time > time) {
        prev = this.keyframes[i];
        next = this.keyframes[i + 1];
        break;
      }
    }

    // Interpolate between keyframes
    const t = (time - prev.time) / (next.time - prev.time);
    const easedT = next.easing(t);

    return this.interpolateStates(prev.state, next.state, easedT);
  }
}
```

## Pattern: Asset Loading Strategy

### Good Example: Predictive Loading
```typescript
// ✅ Load cutscene assets intelligently
class CutsceneAssetManager {
  private loadedAssets: Map<string, any> = new Map();
  private loadingPromises: Map<string, Promise<any>> = new Map();

  async preloadCutscene(cutsceneId: string): Promise<void> {
    const manifest = this.getCutsceneManifest(cutsceneId);

    // Load assets in priority order
    const highPriority = manifest.assets.filter(a => a.priority === 'high');
    const lowPriority = manifest.assets.filter(a => a.priority === 'low');

    // Load high priority assets first
    await Promise.all(highPriority.map(asset => this.loadAsset(asset)));

    // Load low priority in background
    lowPriority.forEach(asset => this.loadAsset(asset));
  }

  private async loadAsset(asset: AssetDescriptor): Promise<any> {
    if (this.loadedAssets.has(asset.id)) {
      return this.loadedAssets.get(asset.id);
    }

    if (this.loadingPromises.has(asset.id)) {
      return this.loadingPromises.get(asset.id);
    }

    const loadPromise = this.loadAssetByType(asset);
    this.loadingPromises.set(asset.id, loadPromise);

    try {
      const loaded = await loadPromise;
      this.loadedAssets.set(asset.id, loaded);
      this.loadingPromises.delete(asset.id);
      return loaded;
    } catch (error) {
      this.loadingPromises.delete(asset.id);
      throw error;
    }
  }

  unloadCutscene(cutsceneId: string): void {
    const manifest = this.getCutsceneManifest(cutsceneId);

    for (const asset of manifest.assets) {
      if (asset.canUnload && !this.isAssetInUse(asset.id)) {
        this.unloadAsset(asset.id);
      }
    }
  }
}

// Predictive preloading based on game progress
class PredictiveCutsceneLoader {
  private predictions: Map<string, number> = new Map();

  update(player: Player): void {
    // Calculate probability of upcoming cutscenes
    for (const [cutsceneId, triggers] of this.cutsceneTriggers) {
      const probability = this.calculateProbability(player, triggers);
      this.predictions.set(cutsceneId, probability);
    }

    // Preload high probability cutscenes
    for (const [cutsceneId, probability] of this.predictions) {
      if (probability > 0.7) {
        this.assetManager.preloadCutscene(cutsceneId);
      } else if (probability < 0.3) {
        this.assetManager.unloadCutscene(cutsceneId);
      }
    }
  }

  private calculateProbability(
    player: Player,
    triggers: CutsceneTrigger[]
  ): number {
    // Calculate based on player position, quest progress, etc.
    let maxProbability = 0;

    for (const trigger of triggers) {
      if (trigger.type === 'proximity') {
        const distance = player.position.distanceTo(trigger.position);
        const probability = Math.max(0, 1 - distance / trigger.range);
        maxProbability = Math.max(maxProbability, probability);
      } else if (trigger.type === 'quest') {
        const progress = player.questProgress.get(trigger.questId) ?? 0;
        const probability = progress / trigger.requiredProgress;
        maxProbability = Math.max(maxProbability, probability);
      }
    }

    return maxProbability;
  }
}
```

## Pattern: Dialogue System Integration

### Good Example: Flexible Dialogue During Cutscenes
```typescript
// ✅ Rich dialogue system that works with cutscenes
interface DialogueNode {
  id: string;
  speaker: string;
  text: string;
  choices?: DialogueChoice[];
  conditions?: DialogueCondition[];
  effects?: DialogueEffect[];
  animation?: string;
  cameraOverride?: CameraState;
}

class CutsceneDialogueController {
  private currentNode: DialogueNode | null = null;
  private dialogueQueue: DialogueNode[] = [];
  private skipBuffer: number = 0; // Prevent accidental skips

  async playDialogue(dialogue: DialogueNode[]): Promise<void> {
    this.dialogueQueue = [...dialogue];

    while (this.dialogueQueue.length > 0) {
      const node = this.dialogueQueue.shift()!;

      if (!this.checkConditions(node.conditions)) {
        continue;
      }

      await this.displayNode(node);

      if (node.choices && node.choices.length > 0) {
        await this.handleChoice(node);
      }

      this.applyEffects(node.effects);
    }
  }

  private async displayNode(node: DialogueNode): Promise<void> {
    this.currentNode = node;

    // Camera override for this line
    if (node.cameraOverride) {
      camera.overrideTemporary(node.cameraOverride);
    }

    // Character animation
    if (node.animation) {
      const character = this.getCharacter(node.speaker);
      character?.playAnimation(node.animation);
    }

    // Display dialogue
    ui.dialogue.show({
      speaker: node.speaker,
      text: node.text,
      typewriter: this.shouldUseTypewriter()
    });

    // Wait for player input or auto-advance
    await this.waitForAdvance();
  }

  private async waitForAdvance(): Promise<void> {
    return new Promise((resolve) => {
      const autoAdvanceTime = this.calculateAutoAdvanceTime(this.currentNode!.text);
      let autoAdvanceTimer: number | null = null;

      if (settings.autoAdvance) {
        autoAdvanceTimer = setTimeout(() => resolve(), autoAdvanceTime);
      }

      const handleInput = (action: string) => {
        if (action === 'advance' && performance.now() > this.skipBuffer) {
          if (autoAdvanceTimer) clearTimeout(autoAdvanceTimer);
          input.off('action', handleInput);
          resolve();
        }
      };

      input.on('action', handleInput);
    });
  }

  private async handleChoice(node: DialogueNode): Promise<void> {
    const availableChoices = node.choices!.filter(choice =>
      this.checkConditions(choice.conditions)
    );

    if (availableChoices.length === 0) return;

    const selectedIndex = await ui.dialogue.promptChoice(
      availableChoices.map(c => ({
        text: c.text,
        preview: c.preview
      }))
    );

    const selected = availableChoices[selectedIndex];

    // Add resulting dialogue to queue
    if (selected.nextDialogue) {
      this.dialogueQueue.unshift(...selected.nextDialogue);
    }

    // Track choice for branching
    gameState.recordChoice(node.id, selected.id);
  }
}
```

## Best Practices

1. **Use state machines** - Avoid boolean flag chaos
2. **Implement skip functionality** - But protect critical gameplay moments
3. **Blend animations smoothly** - Jarring transitions break immersion
4. **Preload assets** - Don't make players wait when cutscene starts
5. **Handle interruptions gracefully** - Save state if player quits mid-cutscene
6. **Test all branches** - Especially choice-based cutscenes
7. **Profile performance** - Cutscenes shouldn't drop frames
8. **Support accessibility** - Subtitles, pause, speed controls
9. **Clean up properly** - Reset all systems after cutscene
10. **Version cutscene data** - For save game compatibility

## Testing Cutscenes

```typescript
describe('CutsceneSystem', () => {
  it('should transition states correctly', () => {
    const fsm = new GameStateMachine();

    expect(fsm.transition('cutscene_starting')).toBe(true);
    expect(fsm.getCurrentState()).toBe('cutscene_starting');
    expect(gameInput.isEnabled()).toBe(false);
  });

  it('should handle skip during non-critical moments', async () => {
    const cutscene = new Cutscene({ skippable: true });
    const promise = cutscene.play();

    await wait(100);
    cutscene.skip();

    await expect(promise).resolves.not.toThrow();
    expect(cutscene.wasSkipped()).toBe(true);
  });

  it('should blend cameras smoothly', () => {
    const controller = new CinematicCameraController();
    const initial = camera.getState();

    controller.startCinematic('test', 1000);
    controller.update(500); // Half blend

    const current = camera.getState();
    expect(current.position).not.toEqual(initial.position);
    expect(controller.getBlendValue()).toBeCloseTo(0.5, 1);
  });
});
```

Remember: The best cutscene is one that enhances the game experience without breaking the flow of gameplay.