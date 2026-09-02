# Conditional Logic Patterns in Game Development

## Managing Complex Game State Conditions

Game development requires managing intricate conditional logic for features like quest progression, item availability, skill unlocks, and world state changes. This document explores effective patterns and anti-patterns using TypeScript examples.

## Anti-Pattern: Nested If-Statement Hell

### Bad Example
```typescript
// ❌ Deeply nested, hard to read, difficult to maintain
function canAccessSecretArea(player: Player): boolean {
  if (player.level >= 10) {
    if (player.inventory.has('mystical_key')) {
      if (player.questLog.isComplete('ancient_prophecy')) {
        if (player.stats.wisdom >= 15) {
          if (!player.debuffs.has('cursed')) {
            if (gameTime.currentPhase === 'night') {
              return true;
            }
          }
        }
      }
    }
  }
  return false;
}
```

### Good Example: Early Returns
```typescript
// ✅ Flat structure, clear requirements, easy to modify
function canAccessSecretArea(player: Player): boolean {
  if (player.level < 10) return false;
  if (!player.inventory.has('mystical_key')) return false;
  if (!player.questLog.isComplete('ancient_prophecy')) return false;
  if (player.stats.wisdom < 15) return false;
  if (player.debuffs.has('cursed')) return false;
  if (gameTime.currentPhase !== 'night') return false;

  return true;
}
```

## Pattern: Predicate System

### Bad Example: Hardcoded Conditions
```typescript
// ❌ Conditions are scattered, hard to reuse, prone to inconsistency
class QuestManager {
  canStartDragonQuest(player: Player): boolean {
    return player.level >= 20 &&
           player.inventory.has('dragon_scale') &&
           player.reputation.get('knights_guild') >= 100;
  }

  canStartGoblinQuest(player: Player): boolean {
    return player.level >= 5 &&
           player.gold >= 50 &&
           !player.questLog.isActive('dragon_quest');
  }
}
```

### Good Example: Composable Predicates
```typescript
// ✅ Reusable, testable, composable conditions
interface Predicate<T> {
  evaluate(context: T): boolean;
  describe(): string;
}

class MinLevelPredicate implements Predicate<Player> {
  constructor(private minLevel: number) {}

  evaluate(player: Player): boolean {
    return player.level >= this.minLevel;
  }

  describe(): string {
    return `Requires level ${this.minLevel}`;
  }
}

class HasItemPredicate implements Predicate<Player> {
  constructor(private itemId: string) {}

  evaluate(player: Player): boolean {
    return player.inventory.has(this.itemId);
  }

  describe(): string {
    return `Requires item: ${this.itemId}`;
  }
}

class CompositePredicate<T> implements Predicate<T> {
  constructor(
    private predicates: Predicate<T>[],
    private mode: 'AND' | 'OR' = 'AND'
  ) {}

  evaluate(context: T): boolean {
    if (this.mode === 'AND') {
      return this.predicates.every(p => p.evaluate(context));
    }
    return this.predicates.some(p => p.evaluate(context));
  }

  describe(): string {
    const descriptions = this.predicates.map(p => p.describe());
    return descriptions.join(this.mode === 'AND' ? ' AND ' : ' OR ');
  }
}

// Usage
const dragonQuestRequirements = new CompositePredicate([
  new MinLevelPredicate(20),
  new HasItemPredicate('dragon_scale'),
  new MinReputationPredicate('knights_guild', 100)
]);

const canStart = dragonQuestRequirements.evaluate(player);
const requirements = dragonQuestRequirements.describe();
```

## Pattern: State Machine for Complex Flows

### Bad Example: Boolean Flags Chaos
```typescript
// ❌ State is implicit, easy to create invalid states
class Door {
  isLocked: boolean = true;
  isOpen: boolean = false;
  isBroken: boolean = false;
  hasBeenLockpicked: boolean = false;

  tryOpen(): boolean {
    if (this.isBroken) {
      return false; // Can't open broken door
    }
    if (this.isOpen) {
      return true; // Already open
    }
    if (this.isLocked && !this.hasBeenLockpicked) {
      return false; // Locked
    }
    this.isOpen = true;
    return true;
  }

  // Invalid state possible: isOpen=true AND isLocked=true
}
```

### Good Example: Explicit State Machine
```typescript
// ✅ Explicit states, impossible to have invalid combinations
type DoorState = 'locked' | 'unlocked' | 'open' | 'broken';

class Door {
  private state: DoorState = 'locked';

  private transitions: Record<DoorState, Partial<Record<DoorAction, DoorState>>> = {
    locked: {
      unlock: 'unlocked',
      lockpick: 'unlocked',
      break: 'broken'
    },
    unlocked: {
      open: 'open',
      lock: 'locked',
      break: 'broken'
    },
    open: {
      close: 'unlocked',
      break: 'broken'
    },
    broken: {} // No transitions from broken state
  };

  performAction(action: DoorAction): boolean {
    const nextState = this.transitions[this.state][action];
    if (nextState) {
      this.state = nextState;
      return true;
    }
    return false;
  }

  canPerformAction(action: DoorAction): boolean {
    return this.transitions[this.state][action] !== undefined;
  }
}
```

## Pattern: Rule-Based Systems

### Bad Example: Hardcoded Combat Rules
```typescript
// ❌ Every new condition requires modifying core logic
function calculateDamage(attacker: Unit, defender: Unit, baseAmount: number): number {
  let damage = baseAmount;

  if (attacker.hasAbility('strength_boost')) {
    damage *= 1.5;
  }
  if (defender.hasAbility('armor_master')) {
    damage *= 0.7;
  }
  if (attacker.weapon?.type === 'fire' && defender.weakness === 'fire') {
    damage *= 2;
  }
  if (gameTime.isNight && attacker.race === 'vampire') {
    damage *= 1.3;
  }
  // Adding new conditions requires modifying this function

  return damage;
}
```

### Good Example: Extensible Rule System
```typescript
// ✅ New rules can be added without modifying existing code
interface DamageRule {
  priority: number;
  applies(context: CombatContext): boolean;
  modify(damage: number, context: CombatContext): number;
}

class ElementalWeaknessRule implements DamageRule {
  priority = 10;

  applies(context: CombatContext): boolean {
    return context.attacker.weapon?.element !== undefined &&
           context.defender.weaknesses.includes(context.attacker.weapon.element);
  }

  modify(damage: number, context: CombatContext): number {
    return damage * 2;
  }
}

class NightVampireRule implements DamageRule {
  priority = 5;

  applies(context: CombatContext): boolean {
    return gameTime.isNight && context.attacker.race === 'vampire';
  }

  modify(damage: number): number {
    return damage * 1.3;
  }
}

class DamageCalculator {
  private rules: DamageRule[] = [];

  addRule(rule: DamageRule): void {
    this.rules.push(rule);
    this.rules.sort((a, b) => b.priority - a.priority);
  }

  calculate(context: CombatContext): number {
    let damage = context.baseDamage;

    for (const rule of this.rules) {
      if (rule.applies(context)) {
        damage = rule.modify(damage, context);
      }
    }

    return damage;
  }
}
```

## Pattern: Requirement Trees

### Bad Example: Flat Skill Requirements
```typescript
// ❌ Can't express complex dependencies
interface SkillRequirement {
  skillId: string;
  requiredLevel: number;
  requiredSkills: string[]; // Just IDs, no structure
}

function canLearnSkill(player: Player, requirement: SkillRequirement): boolean {
  if (player.level < requirement.requiredLevel) return false;

  for (const skillId of requirement.requiredSkills) {
    if (!player.skills.has(skillId)) return false;
  }

  return true;
}
```

### Good Example: Hierarchical Requirements
```typescript
// ✅ Can express complex AND/OR relationships
abstract class Requirement {
  abstract isMet(player: Player): boolean;
  abstract getMissingRequirements(player: Player): string[];
}

class LevelRequirement extends Requirement {
  constructor(private minLevel: number) {
    super();
  }

  isMet(player: Player): boolean {
    return player.level >= this.minLevel;
  }

  getMissingRequirements(player: Player): string[] {
    if (this.isMet(player)) return [];
    return [`Need level ${this.minLevel} (current: ${player.level})`];
  }
}

class SkillRequirement extends Requirement {
  constructor(private skillId: string, private minRank: number = 1) {
    super();
  }

  isMet(player: Player): boolean {
    const rank = player.skills.get(this.skillId)?.rank ?? 0;
    return rank >= this.minRank;
  }

  getMissingRequirements(player: Player): string[] {
    if (this.isMet(player)) return [];
    return [`Need ${this.skillId} rank ${this.minRank}`];
  }
}

class CompositeRequirement extends Requirement {
  constructor(
    private requirements: Requirement[],
    private operator: 'AND' | 'OR' | 'NOT' = 'AND'
  ) {
    super();
  }

  isMet(player: Player): boolean {
    switch (this.operator) {
      case 'AND':
        return this.requirements.every(req => req.isMet(player));
      case 'OR':
        return this.requirements.some(req => req.isMet(player));
      case 'NOT':
        return !this.requirements[0]?.isMet(player);
    }
  }

  getMissingRequirements(player: Player): string[] {
    if (this.isMet(player)) return [];

    const missing: string[] = [];
    for (const req of this.requirements) {
      missing.push(...req.getMissingRequirements(player));
    }
    return missing;
  }
}

// Complex skill tree example
const fireball2Requirements = new CompositeRequirement([
  new LevelRequirement(10),
  new CompositeRequirement([
    new SkillRequirement('fireball', 5),
    new CompositeRequirement([
      new SkillRequirement('fire_affinity', 3),
      new SkillRequirement('mana_control', 2)
    ], 'OR')
  ])
]);
```

## Pattern: Event-Driven Conditions

### Bad Example: Polling State
```typescript
// ❌ Checking conditions repeatedly, performance issues
class QuestManager {
  private quests: Quest[] = [];

  update(): void {
    // Called every frame!
    for (const quest of this.quests) {
      if (!quest.isActive && this.checkQuestRequirements(quest)) {
        this.activateQuest(quest);
      }

      if (quest.isActive && this.checkQuestCompletion(quest)) {
        this.completeQuest(quest);
      }
    }
  }
}
```

### Good Example: Event-Based Triggers
```typescript
// ✅ Only check when relevant events occur
class QuestManager {
  private questTriggers = new Map<string, Quest[]>();

  registerQuest(quest: Quest): void {
    for (const trigger of quest.triggers) {
      if (!this.questTriggers.has(trigger)) {
        this.questTriggers.set(trigger, []);
      }
      this.questTriggers.get(trigger)!.push(quest);
    }
  }

  onEvent(event: GameEvent): void {
    const relevantQuests = this.questTriggers.get(event.type) ?? [];

    for (const quest of relevantQuests) {
      if (!quest.isActive && quest.shouldActivate(event)) {
        this.activateQuest(quest);
      }

      if (quest.isActive && quest.shouldProgress(event)) {
        quest.progress(event);
      }
    }
  }
}

// Usage
eventBus.on('item_collected', (event) => questManager.onEvent(event));
eventBus.on('enemy_defeated', (event) => questManager.onEvent(event));
eventBus.on('area_entered', (event) => questManager.onEvent(event));
```

## Pattern: Condition Caching

### Bad Example: Recalculating Everything
```typescript
// ❌ Expensive calculations repeated unnecessarily
class Player {
  canCastSpell(spell: Spell): boolean {
    const totalMana = this.calculateTotalMana(); // Expensive
    const spellCost = spell.calculateCost(this); // Expensive
    const hasRequirements = spell.requirements.every(req =>
      this.meetsRequirement(req) // Expensive
    );

    return totalMana >= spellCost && hasRequirements;
  }
}
```

### Good Example: Smart Caching
```typescript
// ✅ Cache results and invalidate when necessary
class Player {
  private cache = new Map<string, { value: any; dependencies: Set<string> }>();

  private getCached<T>(key: string, calculator: () => T, dependencies: string[]): T {
    const cached = this.cache.get(key);

    if (cached && this.areDependenciesValid(cached.dependencies)) {
      return cached.value;
    }

    const value = calculator();
    this.cache.set(key, {
      value,
      dependencies: new Set(dependencies)
    });

    return value;
  }

  invalidateCache(...keys: string[]): void {
    for (const key of keys) {
      this.cache.delete(key);
    }

    // Also invalidate dependent caches
    for (const [cacheKey, cached] of this.cache.entries()) {
      if (keys.some(key => cached.dependencies.has(key))) {
        this.cache.delete(cacheKey);
      }
    }
  }

  get totalMana(): number {
    return this.getCached('totalMana',
      () => this.calculateTotalMana(),
      ['equipment', 'buffs', 'stats']
    );
  }

  onEquipmentChange(): void {
    this.invalidateCache('equipment');
  }
}
```

## Best Practices Summary

1. **Keep conditions flat** - Use early returns instead of nested ifs
2. **Make states explicit** - Use enums/types instead of boolean combinations
3. **Compose small predicates** - Build complex conditions from simple, testable units
4. **Use event-driven updates** - Don't poll state continuously
5. **Cache expensive calculations** - But invalidate correctly
6. **Separate concerns** - Rules, requirements, and state should be independent
7. **Make illegal states unrepresentable** - Use type systems to prevent invalid combinations
8. **Document complex conditions** - Future you will thank present you

## Testing Conditional Logic

```typescript
// Good: Test individual predicates
describe('MinLevelPredicate', () => {
  it('should return true when player level meets requirement', () => {
    const predicate = new MinLevelPredicate(10);
    const player = createTestPlayer({ level: 10 });
    expect(predicate.evaluate(player)).toBe(true);
  });

  it('should return false when player level is too low', () => {
    const predicate = new MinLevelPredicate(10);
    const player = createTestPlayer({ level: 9 });
    expect(predicate.evaluate(player)).toBe(false);
  });
});

// Good: Test composite conditions
describe('Quest Requirements', () => {
  it('should require all conditions for AND composite', () => {
    const requirements = new CompositePredicate([
      new MinLevelPredicate(5),
      new HasItemPredicate('key')
    ], 'AND');

    const playerWithoutKey = createTestPlayer({ level: 5 });
    expect(requirements.evaluate(playerWithoutKey)).toBe(false);

    const playerWithKey = createTestPlayer({
      level: 5,
      inventory: ['key']
    });
    expect(requirements.evaluate(playerWithKey)).toBe(true);
  });
});
```

Remember: The goal isn't to eliminate complexity, but to manage it in a maintainable, testable, and understandable way.