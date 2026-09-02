# Save System Architecture in Games

## Data Structures and Serialization Patterns

This document explores technical approaches to implementing robust save/load systems in games, with TypeScript examples demonstrating various architectural patterns.

## Core Challenges

1. **Data Versioning** - Save files must work across game updates
2. **Performance** - Large worlds need efficient serialization
3. **Integrity** - Corrupted saves must be detected/recovered
4. **Platform Compatibility** - Different systems have different requirements
5. **State Consistency** - All game systems must save/load coherently

## Anti-Pattern: Monolithic Save Object

### Bad Example
```typescript
// ❌ Everything in one massive object, hard to version, memory intensive
interface SaveGame {
  version: number;
  playerName: string;
  playerLevel: number;
  playerHealth: number;
  playerMana: number;
  playerX: number;
  playerY: number;
  playerZ: number;
  inventory: Item[];
  quests: Quest[];
  npcs: NPC[];
  worldObjects: GameObject[];
  weatherState: Weather;
  timeOfDay: number;
  // ... hundreds more fields
}

function saveGame(game: GameState): void {
  const save: SaveGame = {
    version: 1,
    playerName: game.player.name,
    playerLevel: game.player.level,
    // ... manually copy every field
  };

  fs.writeFileSync('save.json', JSON.stringify(save));
}
```

## Pattern: Modular Save System

### Good Example: Component-Based Serialization
```typescript
// ✅ Each system handles its own serialization
interface Saveable {
  getSerializationId(): string;
  serialize(): unknown;
  deserialize(data: unknown): void;
}

interface SaveData {
  version: string;
  timestamp: number;
  modules: Map<string, unknown>;
}

class SaveManager {
  private saveables: Map<string, Saveable> = new Map();

  register(saveable: Saveable): void {
    this.saveables.set(saveable.getSerializationId(), saveable);
  }

  save(): SaveData {
    const data: SaveData = {
      version: '1.2.0',
      timestamp: Date.now(),
      modules: new Map()
    };

    for (const [id, saveable] of this.saveables) {
      try {
        data.modules.set(id, saveable.serialize());
      } catch (error) {
        console.error(`Failed to serialize ${id}:`, error);
      }
    }

    return data;
  }

  load(data: SaveData): void {
    for (const [id, moduleData] of data.modules) {
      const saveable = this.saveables.get(id);
      if (saveable) {
        try {
          saveable.deserialize(moduleData);
        } catch (error) {
          console.error(`Failed to deserialize ${id}:`, error);
        }
      }
    }
  }
}

// Example implementation
class PlayerSystem implements Saveable {
  private player: Player;

  getSerializationId(): string {
    return 'player_v1';
  }

  serialize(): unknown {
    return {
      name: this.player.name,
      level: this.player.level,
      position: {
        x: this.player.position.x,
        y: this.player.position.y,
        z: this.player.position.z
      },
      stats: this.player.stats.toJSON()
    };
  }

  deserialize(data: unknown): void {
    const playerData = data as any;
    this.player.name = playerData.name;
    this.player.level = playerData.level;
    this.player.position.set(
      playerData.position.x,
      playerData.position.y,
      playerData.position.z
    );
    this.player.stats.fromJSON(playerData.stats);
  }
}
```

## Pattern: Versioned Serialization

### Bad Example: No Version Management
```typescript
// ❌ Breaks when data structure changes
class Inventory {
  items: Item[] = [];

  serialize(): any {
    return this.items; // What if Item structure changes?
  }

  deserialize(data: any): void {
    this.items = data; // Will crash if old format
  }
}
```

### Good Example: Migration System
```typescript
// ✅ Handles version differences gracefully
interface VersionedData {
  version: number;
  data: unknown;
}

abstract class VersionedSerializer<T> {
  abstract currentVersion: number;
  abstract migrations: Map<number, (data: any) => any>;

  serialize(obj: T): VersionedData {
    return {
      version: this.currentVersion,
      data: this.serializeLatest(obj)
    };
  }

  deserialize(versionedData: VersionedData): T {
    let data = versionedData.data;
    let version = versionedData.version;

    // Apply migrations to bring data to current version
    while (version < this.currentVersion) {
      const migration = this.migrations.get(version);
      if (!migration) {
        throw new Error(`No migration from version ${version}`);
      }
      data = migration(data);
      version++;
    }

    return this.deserializeLatest(data);
  }

  protected abstract serializeLatest(obj: T): unknown;
  protected abstract deserializeLatest(data: unknown): T;
}

class InventorySerializer extends VersionedSerializer<Inventory> {
  currentVersion = 3;

  migrations = new Map([
    // Version 1 -> 2: Added item durability
    [1, (data: any) => {
      return data.map((item: any) => ({
        ...item,
        durability: 100 // Default durability for old items
      }));
    }],
    // Version 2 -> 3: Changed weight from number to object
    [2, (data: any) => {
      return data.map((item: any) => ({
        ...item,
        weight: {
          value: item.weight,
          unit: 'kg'
        }
      }));
    }]
  ]);

  protected serializeLatest(inventory: Inventory): unknown {
    return inventory.items.map(item => ({
      id: item.id,
      quantity: item.quantity,
      durability: item.durability,
      weight: item.weight,
      metadata: item.metadata
    }));
  }

  protected deserializeLatest(data: unknown): Inventory {
    const inventory = new Inventory();
    const items = data as any[];
    inventory.items = items.map(itemData =>
      new Item(itemData.id, itemData.quantity, itemData)
    );
    return inventory;
  }
}
```

## Pattern: Differential Saves

### Bad Example: Full Saves Every Time
```typescript
// ❌ Wastes space and time saving unchanged data
class World {
  private chunks: Map<string, Chunk> = new Map();

  save(): void {
    const allChunks = Array.from(this.chunks.values());
    // Save ALL chunks even if 99% haven't changed
    fs.writeFileSync('world.dat', serialize(allChunks));
  }
}
```

### Good Example: Delta-Based Saves
```typescript
// ✅ Only save what changed
interface SaveDelta {
  baseVersion: number;
  changes: Change[];
}

interface Change {
  type: 'create' | 'update' | 'delete';
  path: string[];
  value?: any;
  timestamp: number;
}

class DifferentialSaveManager {
  private baseSave: SaveData | null = null;
  private changes: Change[] = [];
  private saveInterval = 5; // Full save every 5 deltas

  trackChange(path: string[], type: 'create' | 'update' | 'delete', value?: any): void {
    this.changes.push({
      type,
      path,
      value,
      timestamp: Date.now()
    });
  }

  save(force: boolean = false): void {
    if (!this.baseSave || this.changes.length >= this.saveInterval || force) {
      // Full save
      this.baseSave = this.createFullSave();
      this.changes = [];
      this.writeFullSave(this.baseSave);
    } else {
      // Delta save
      const delta: SaveDelta = {
        baseVersion: this.baseSave.version,
        changes: this.changes
      };
      this.writeDeltaSave(delta);
    }
  }

  load(): SaveData {
    const baseSave = this.readBaseSave();
    const deltas = this.readDeltaSaves();

    let current = baseSave;
    for (const delta of deltas) {
      current = this.applyDelta(current, delta);
    }

    return current;
  }

  private applyDelta(save: SaveData, delta: SaveDelta): SaveData {
    const result = structuredClone(save);

    for (const change of delta.changes) {
      switch (change.type) {
        case 'create':
        case 'update':
          this.setNestedValue(result, change.path, change.value);
          break;
        case 'delete':
          this.deleteNestedValue(result, change.path);
          break;
      }
    }

    return result;
  }
}
```

## Pattern: Compression and Optimization

### Good Example: Smart Compression
```typescript
// ✅ Reduce save file size intelligently
class CompressedSaveManager {
  private compressionStrategies = new Map<string, CompressionStrategy>();

  save(data: SaveData): Buffer {
    const sections: Buffer[] = [];

    for (const [key, value] of data.modules) {
      const strategy = this.compressionStrategies.get(key) || new DefaultCompression();
      const compressed = strategy.compress(value);
      sections.push(compressed);
    }

    return Buffer.concat(sections);
  }
}

interface CompressionStrategy {
  compress(data: unknown): Buffer;
  decompress(buffer: Buffer): unknown;
}

class PositionArrayCompression implements CompressionStrategy {
  compress(data: unknown): Buffer {
    const positions = data as Array<{x: number, y: number, z: number}>;
    // Store as Float32Array for better compression
    const floatArray = new Float32Array(positions.length * 3);

    positions.forEach((pos, i) => {
      floatArray[i * 3] = pos.x;
      floatArray[i * 3 + 1] = pos.y;
      floatArray[i * 3 + 2] = pos.z;
    });

    // Further compress with zlib
    return zlib.deflateSync(Buffer.from(floatArray.buffer));
  }

  decompress(buffer: Buffer): unknown {
    const decompressed = zlib.inflateSync(buffer);
    const floatArray = new Float32Array(decompressed.buffer);
    const positions = [];

    for (let i = 0; i < floatArray.length; i += 3) {
      positions.push({
        x: floatArray[i],
        y: floatArray[i + 1],
        z: floatArray[i + 2]
      });
    }

    return positions;
  }
}

// Sparse data optimization
class SparseWorldSave {
  serialize(world: World): unknown {
    const modifiedChunks = new Map<string, any>();

    for (const [key, chunk] of world.chunks) {
      if (chunk.isModified) {
        // Only save chunks that differ from procedural generation
        const changes = chunk.getDifferenceFromDefault();
        if (changes.length > 0) {
          modifiedChunks.set(key, changes);
        }
      }
    }

    return {
      seed: world.seed,
      modifiedChunks: Object.fromEntries(modifiedChunks)
    };
  }
}
```

## Pattern: Save File Integrity

### Good Example: Checksums and Recovery
```typescript
// ✅ Detect and handle corruption
interface SecureSaveFile {
  header: {
    version: string;
    timestamp: number;
    checksum: string;
  };
  data: Buffer;
  backup?: Buffer;
}

class SecureSaveManager {
  private readonly maxBackups = 3;

  async save(data: SaveData): Promise<void> {
    const serialized = this.serialize(data);
    const checksum = this.calculateChecksum(serialized);

    const saveFile: SecureSaveFile = {
      header: {
        version: '1.0.0',
        timestamp: Date.now(),
        checksum
      },
      data: serialized
    };

    // Rotate backups
    await this.rotateBackups();

    // Write with atomic operation
    await this.atomicWrite('save.dat', saveFile);
  }

  async load(): Promise<SaveData | null> {
    try {
      const saveFile = await this.readSaveFile('save.dat');

      if (this.validateChecksum(saveFile)) {
        return this.deserialize(saveFile.data);
      }

      console.warn('Save file corrupted, attempting recovery...');
      return await this.attemptRecovery();
    } catch (error) {
      console.error('Failed to load save:', error);
      return await this.loadBackup();
    }
  }

  private async atomicWrite(path: string, data: SecureSaveFile): Promise<void> {
    const tempPath = `${path}.tmp`;

    // Write to temp file first
    await fs.promises.writeFile(tempPath, this.encodeSaveFile(data));

    // Atomic rename
    await fs.promises.rename(tempPath, path);
  }

  private async attemptRecovery(): Promise<SaveData | null> {
    // Try to load backups in order
    for (let i = 0; i < this.maxBackups; i++) {
      try {
        const backup = await this.loadBackup(i);
        if (backup) {
          console.log(`Recovered from backup ${i}`);
          return backup;
        }
      } catch (error) {
        continue;
      }
    }
    return null;
  }

  private calculateChecksum(data: Buffer): string {
    return crypto.createHash('sha256').update(data).digest('hex');
  }

  private validateChecksum(saveFile: SecureSaveFile): boolean {
    const calculated = this.calculateChecksum(saveFile.data);
    return calculated === saveFile.header.checksum;
  }
}
```

## Pattern: Async Save Operations

### Good Example: Non-Blocking Saves
```typescript
// ✅ Save without freezing the game
class AsyncSaveManager {
  private saveQueue: SaveRequest[] = [];
  private isSaving = false;

  async requestSave(priority: 'high' | 'normal' | 'low' = 'normal'): Promise<void> {
    return new Promise((resolve, reject) => {
      const request: SaveRequest = {
        priority,
        resolve,
        reject,
        timestamp: Date.now()
      };

      this.saveQueue.push(request);
      this.saveQueue.sort((a, b) => {
        const priorityOrder = { high: 0, normal: 1, low: 2 };
        return priorityOrder[a.priority] - priorityOrder[b.priority];
      });

      this.processSaveQueue();
    });
  }

  private async processSaveQueue(): Promise<void> {
    if (this.isSaving || this.saveQueue.length === 0) {
      return;
    }

    this.isSaving = true;
    const request = this.saveQueue.shift()!;

    try {
      // Collect save data in main thread
      const data = await this.collectSaveData();

      // Serialize in worker thread
      const serialized = await this.serializeInWorker(data);

      // Write to disk async
      await this.writeAsync(serialized);

      request.resolve();
    } catch (error) {
      request.reject(error);
    } finally {
      this.isSaving = false;

      // Process next request
      if (this.saveQueue.length > 0) {
        setImmediate(() => this.processSaveQueue());
      }
    }
  }

  private async serializeInWorker(data: SaveData): Promise<Buffer> {
    return new Promise((resolve, reject) => {
      const worker = new Worker('./save-serializer.worker.js');

      worker.postMessage(data);
      worker.once('message', (serialized: Buffer) => {
        worker.terminate();
        resolve(serialized);
      });
      worker.once('error', reject);
    });
  }
}
```

## Platform-Specific Considerations

```typescript
// Platform abstraction
interface PlatformSaveAdapter {
  getMaxSaveSize(): number;
  getSavePath(): string;
  requiresCloudSync(): boolean;
  write(path: string, data: Buffer): Promise<void>;
  read(path: string): Promise<Buffer>;
}

class SteamSaveAdapter implements PlatformSaveAdapter {
  getMaxSaveSize(): number {
    return 100 * 1024 * 1024; // 100MB Steam Cloud limit
  }

  getSavePath(): string {
    return steamworks.getCloudSavePath();
  }

  requiresCloudSync(): boolean {
    return true;
  }

  async write(path: string, data: Buffer): Promise<void> {
    await steamworks.writeCloudFile(path, data);
  }

  async read(path: string): Promise<Buffer> {
    return steamworks.readCloudFile(path);
  }
}

class PlatformAwareSaveManager {
  constructor(private adapter: PlatformSaveAdapter) {}

  async save(data: SaveData): Promise<void> {
    const serialized = this.serialize(data);

    if (serialized.length > this.adapter.getMaxSaveSize()) {
      throw new Error('Save file too large for platform');
    }

    await this.adapter.write('game.sav', serialized);

    if (this.adapter.requiresCloudSync()) {
      await this.syncToCloud();
    }
  }
}
```

## Best Practices

1. **Always version your save format** - You will need to change it
2. **Use compression wisely** - Balance file size vs. CPU usage
3. **Implement integrity checks** - Players hate corrupted saves
4. **Make saves atomic** - Never partially write a save
5. **Keep backups** - Auto-rotate multiple save files
6. **Test save/load extensively** - Including edge cases and corruption
7. **Profile serialization performance** - It impacts gameplay
8. **Document your save format** - Future developers (including you) need this
9. **Handle platform requirements** - Different systems have different limits
10. **Consider save file size** - Cloud sync has limits

## Testing Save Systems

```typescript
describe('SaveSystem', () => {
  it('should handle version migration', async () => {
    const oldSave = createV1Save();
    const manager = new SaveManager();

    await manager.load(oldSave);
    const migrated = await manager.save();

    expect(migrated.version).toBe('2.0.0');
    expect(migrated.data).toHaveProperty('newField');
  });

  it('should recover from corruption', async () => {
    const corruptedSave = Buffer.from('corrupted data');
    const manager = new SecureSaveManager();

    await manager.writeRaw(corruptedSave);
    const recovered = await manager.load();

    expect(recovered).not.toBeNull();
  });

  it('should handle concurrent save requests', async () => {
    const manager = new AsyncSaveManager();

    const saves = Promise.all([
      manager.requestSave('low'),
      manager.requestSave('high'),
      manager.requestSave('normal')
    ]);

    await expect(saves).resolves.not.toThrow();
  });
});
```

Remember: A good save system is invisible when working and invaluable when needed. Players won't notice a great save system, but they'll definitely notice a bad one.