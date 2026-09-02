# Physics Simulation in Games

## Implementation Patterns and Performance Optimization

This document explores technical approaches to implementing physics simulations in games, covering collision detection, rigid body dynamics, and optimization strategies with TypeScript examples.

## Core Physics Components

1. **Collision Detection** - Finding intersections between objects
2. **Collision Response** - Calculating forces and velocities after impact
3. **Integration** - Updating positions and velocities over time
4. **Constraint Solving** - Maintaining joints and connections
5. **Broad Phase Optimization** - Reducing collision checks

## Anti-Pattern: Naive Collision Detection

### Bad Example
```typescript
// ❌ O(n²) complexity, checks every object against every other
class NaivePhysicsWorld {
  objects: PhysicsObject[] = [];

  update(deltaTime: number): void {
    // Check every pair - extremely slow with many objects
    for (let i = 0; i < this.objects.length; i++) {
      for (let j = i + 1; j < this.objects.length; j++) {
        if (this.checkCollision(this.objects[i], this.objects[j])) {
          this.resolveCollision(this.objects[i], this.objects[j]);
        }
      }
    }

    // Update positions
    for (const obj of this.objects) {
      obj.position.x += obj.velocity.x * deltaTime;
      obj.position.y += obj.velocity.y * deltaTime;
      // No consideration for acceleration, forces, etc.
    }
  }
}
```

## Pattern: Spatial Partitioning

### Good Example: Spatial Hash Grid
```typescript
// ✅ Efficient broad phase collision detection
class SpatialHashGrid<T extends PhysicsObject> {
  private grid: Map<string, Set<T>> = new Map();
  private cellSize: number;

  constructor(cellSize: number = 100) {
    this.cellSize = cellSize;
  }

  private hashPosition(x: number, y: number): string {
    const gridX = Math.floor(x / this.cellSize);
    const gridY = Math.floor(y / this.cellSize);
    return `${gridX},${gridY}`;
  }

  private getOccupiedCells(obj: T): string[] {
    const bounds = obj.getBounds();
    const cells: string[] = [];

    const minX = Math.floor(bounds.min.x / this.cellSize);
    const maxX = Math.floor(bounds.max.x / this.cellSize);
    const minY = Math.floor(bounds.min.y / this.cellSize);
    const maxY = Math.floor(bounds.max.y / this.cellSize);

    for (let x = minX; x <= maxX; x++) {
      for (let y = minY; y <= maxY; y++) {
        cells.push(`${x},${y}`);
      }
    }

    return cells;
  }

  insert(obj: T): void {
    const cells = this.getOccupiedCells(obj);

    for (const cell of cells) {
      if (!this.grid.has(cell)) {
        this.grid.set(cell, new Set());
      }
      this.grid.get(cell)!.add(obj);
    }
  }

  update(obj: T, oldBounds: Bounds): void {
    // Remove from old cells
    const oldCells = this.getOccupiedCells({ getBounds: () => oldBounds } as T);
    for (const cell of oldCells) {
      this.grid.get(cell)?.delete(obj);
    }

    // Add to new cells
    this.insert(obj);
  }

  getPotentialCollisions(obj: T): Set<T> {
    const nearby = new Set<T>();
    const cells = this.getOccupiedCells(obj);

    for (const cell of cells) {
      const cellObjects = this.grid.get(cell);
      if (cellObjects) {
        for (const other of cellObjects) {
          if (other !== obj) {
            nearby.add(other);
          }
        }
      }
    }

    return nearby;
  }

  clear(): void {
    this.grid.clear();
  }
}

class OptimizedPhysicsWorld {
  private objects: PhysicsObject[] = [];
  private spatialGrid: SpatialHashGrid<PhysicsObject>;
  private staticGrid: SpatialHashGrid<PhysicsObject>;

  constructor(worldSize: number) {
    const cellSize = worldSize / 20; // Tune based on object sizes
    this.spatialGrid = new SpatialHashGrid(cellSize);
    this.staticGrid = new SpatialHashGrid(cellSize * 2); // Larger cells for static
  }

  update(deltaTime: number): void {
    // Clear dynamic grid (static doesn't need clearing)
    this.spatialGrid.clear();

    // Insert dynamic objects
    for (const obj of this.objects) {
      if (!obj.isStatic) {
        this.spatialGrid.insert(obj);
      }
    }

    // Broad phase + narrow phase
    for (const obj of this.objects) {
      if (obj.isStatic) continue;

      // Get potential collisions from spatial grid
      const potentials = this.spatialGrid.getPotentialCollisions(obj);

      // Also check against static objects
      const staticPotentials = this.staticGrid.getPotentialCollisions(obj);

      // Narrow phase - actual collision detection
      for (const other of [...potentials, ...staticPotentials]) {
        if (this.narrowPhaseCollision(obj, other)) {
          this.resolveCollision(obj, other);
        }
      }
    }

    // Update positions using Verlet integration
    this.integrateMotion(deltaTime);
  }
}
```

## Pattern: Continuous Collision Detection

### Bad Example: Discrete Collision Detection
```typescript
// ❌ Fast objects can tunnel through thin walls
class DiscreteCollisionDetection {
  checkCollision(obj: PhysicsObject, deltaTime: number): void {
    // Just check end position
    obj.position.add(obj.velocity.multiply(deltaTime));

    // Bullet moving at 1000 units/sec can skip through 10 unit wall
    // if deltaTime is > 0.01 seconds
    if (this.isColliding(obj)) {
      // Too late, already through the wall!
    }
  }
}
```

### Good Example: Swept/Continuous Detection
```typescript
// ✅ Detect collisions along entire movement path
class ContinuousCollisionDetection {
  private rayCast(
    start: Vector2,
    end: Vector2,
    obstacle: PhysicsObject
  ): RaycastHit | null {
    const direction = end.subtract(start);
    const distance = direction.length();
    direction.normalize();

    // Check intersection with obstacle bounds
    const bounds = obstacle.getBounds();

    // Ray-AABB intersection
    const invDir = new Vector2(1 / direction.x, 1 / direction.y);
    const t1 = (bounds.min.x - start.x) * invDir.x;
    const t2 = (bounds.max.x - start.x) * invDir.x;
    const t3 = (bounds.min.y - start.y) * invDir.y;
    const t4 = (bounds.max.y - start.y) * invDir.y;

    const tMin = Math.max(Math.min(t1, t2), Math.min(t3, t4));
    const tMax = Math.min(Math.max(t1, t2), Math.max(t3, t4));

    if (tMax < 0 || tMin > tMax || tMin > distance) {
      return null;
    }

    return {
      point: start.add(direction.multiply(tMin)),
      normal: this.calculateNormal(start, direction, bounds),
      distance: tMin,
      object: obstacle
    };
  }

  sweepCollision(
    movingObj: PhysicsObject,
    velocity: Vector2,
    deltaTime: number,
    obstacles: PhysicsObject[]
  ): SweepResult {
    const start = movingObj.position;
    const end = start.add(velocity.multiply(deltaTime));

    let earliestHit: RaycastHit | null = null;
    let earliestTime = 1.0;

    // Expand bounds by object size for accurate sweep
    const expandedBounds = movingObj.getBounds();

    for (const obstacle of obstacles) {
      if (obstacle === movingObj) continue;

      // Minkowski sum - expand obstacle by moving object's size
      const expandedObstacle = this.minkowskiSum(obstacle, expandedBounds);

      const hit = this.rayCast(start, end, expandedObstacle);

      if (hit && hit.distance / end.subtract(start).length() < earliestTime) {
        earliestHit = hit;
        earliestTime = hit.distance / end.subtract(start).length();
      }
    }

    return {
      hit: earliestHit,
      finalPosition: earliestHit
        ? start.add(velocity.multiply(deltaTime * earliestTime * 0.99))
        : end,
      remainingTime: earliestHit ? deltaTime * (1 - earliestTime) : 0
    };
  }
}

// Adaptive CCD - use continuous only for fast objects
class AdaptiveCCD {
  private ccdThreshold: number = 50; // Units per second

  updateObject(obj: PhysicsObject, deltaTime: number): void {
    const speed = obj.velocity.length();

    if (speed > this.ccdThreshold) {
      // Use continuous collision detection
      const sweep = this.sweepCollision(obj, obj.velocity, deltaTime);
      obj.position = sweep.finalPosition;

      if (sweep.hit) {
        this.handleCollision(obj, sweep.hit);
      }
    } else {
      // Use discrete for slow objects
      obj.position.add(obj.velocity.multiply(deltaTime));

      for (const other of this.getNearbyObjects(obj)) {
        if (this.checkOverlap(obj, other)) {
          this.resolveOverlap(obj, other);
        }
      }
    }
  }
}
```

## Pattern: Stable Integration

### Bad Example: Euler Integration
```typescript
// ❌ Accumulates error, unstable at low framerates
class UnstablePhysics {
  update(obj: PhysicsObject, deltaTime: number): void {
    // Euler integration - position depends on current velocity
    obj.position += obj.velocity * deltaTime;
    obj.velocity += obj.acceleration * deltaTime;

    // Energy increases over time (explosion!)
    // Orbit becomes spiral
    // Springs become unstable
  }
}
```

### Good Example: Verlet Integration
```typescript
// ✅ More stable, handles variable timesteps better
class VerletIntegration {
  private previousPositions: Map<PhysicsObject, Vector2> = new Map();

  integrate(obj: PhysicsObject, deltaTime: number): void {
    if (!this.previousPositions.has(obj)) {
      // First frame - use Euler for initialization
      this.previousPositions.set(obj, obj.position.clone());
      obj.position.add(obj.velocity.multiply(deltaTime));
    } else {
      // Verlet integration
      const previousPos = this.previousPositions.get(obj)!;
      const currentPos = obj.position.clone();

      // Calculate new position based on previous positions
      const velocity = currentPos.subtract(previousPos);
      const acceleration = obj.getNetForce().divide(obj.mass);

      obj.position = currentPos
        .add(velocity)
        .add(acceleration.multiply(deltaTime * deltaTime));

      this.previousPositions.set(obj, currentPos);

      // Derive velocity for other systems
      obj.velocity = obj.position.subtract(currentPos).divide(deltaTime);
    }
  }

  // Constraints are easy with Verlet
  applyDistanceConstraint(a: PhysicsObject, b: PhysicsObject, distance: number): void {
    const delta = b.position.subtract(a.position);
    const currentDistance = delta.length();

    if (currentDistance === 0) return;

    const correction = (distance - currentDistance) / currentDistance / 2;
    const correctionVector = delta.multiply(correction);

    if (!a.isStatic) {
      a.position.subtract(correctionVector);
    }
    if (!b.isStatic) {
      b.position.add(correctionVector);
    }
  }
}

// RK4 Integration for high precision
class RK4Integration {
  integrate(obj: PhysicsObject, deltaTime: number): void {
    const k1 = this.evaluate(obj, 0, new Derivative());
    const k2 = this.evaluate(obj, deltaTime * 0.5, k1);
    const k3 = this.evaluate(obj, deltaTime * 0.5, k2);
    const k4 = this.evaluate(obj, deltaTime, k3);

    const dxdt = k1.velocity
      .add(k2.velocity.multiply(2))
      .add(k3.velocity.multiply(2))
      .add(k4.velocity)
      .divide(6);

    const dvdt = k1.acceleration
      .add(k2.acceleration.multiply(2))
      .add(k3.acceleration.multiply(2))
      .add(k4.acceleration)
      .divide(6);

    obj.position.add(dxdt.multiply(deltaTime));
    obj.velocity.add(dvdt.multiply(deltaTime));
  }

  private evaluate(
    obj: PhysicsObject,
    dt: number,
    derivative: Derivative
  ): Derivative {
    const state = {
      position: obj.position.add(derivative.velocity.multiply(dt)),
      velocity: obj.velocity.add(derivative.acceleration.multiply(dt))
    };

    return {
      velocity: state.velocity,
      acceleration: this.calculateAcceleration(state, obj)
    };
  }
}
```

## Pattern: Collision Response

### Good Example: Impulse-Based Response
```typescript
// ✅ Physically accurate collision response
class CollisionResolver {
  resolveCollision(a: PhysicsObject, b: PhysicsObject, manifold: CollisionManifold): void {
    // Calculate relative velocity
    const relativeVelocity = b.velocity.subtract(a.velocity);
    const velocityAlongNormal = relativeVelocity.dot(manifold.normal);

    // Objects moving apart, no resolution needed
    if (velocityAlongNormal > 0) return;

    // Calculate restitution (bounciness)
    const restitution = Math.min(a.restitution, b.restitution);

    // Calculate impulse magnitude
    let impulseMagnitude = -(1 + restitution) * velocityAlongNormal;
    impulseMagnitude /= a.inverseMass + b.inverseMass;

    // Apply impulse
    const impulse = manifold.normal.multiply(impulseMagnitude);

    if (!a.isStatic) {
      a.velocity.subtract(impulse.multiply(a.inverseMass));
    }
    if (!b.isStatic) {
      b.velocity.add(impulse.multiply(b.inverseMass));
    }

    // Apply friction
    this.applyFriction(a, b, manifold, impulseMagnitude);

    // Resolve penetration
    this.resolvePenetration(a, b, manifold);
  }

  private applyFriction(
    a: PhysicsObject,
    b: PhysicsObject,
    manifold: CollisionManifold,
    normalImpulse: number
  ): void {
    const relativeVelocity = b.velocity.subtract(a.velocity);

    // Calculate tangent vector (perpendicular to normal)
    const tangent = relativeVelocity
      .subtract(manifold.normal.multiply(relativeVelocity.dot(manifold.normal)))
      .normalize();

    // Calculate friction impulse
    const velocityAlongTangent = relativeVelocity.dot(tangent);
    const mu = Math.sqrt(a.friction * b.friction);
    let frictionImpulse = -velocityAlongTangent / (a.inverseMass + b.inverseMass);

    // Coulomb friction - clamp to maximum static friction
    frictionImpulse = Math.max(
      -Math.abs(normalImpulse) * mu,
      Math.min(frictionImpulse, Math.abs(normalImpulse) * mu)
    );

    const frictionVector = tangent.multiply(frictionImpulse);

    if (!a.isStatic) {
      a.velocity.subtract(frictionVector.multiply(a.inverseMass));
    }
    if (!b.isStatic) {
      b.velocity.add(frictionVector.multiply(b.inverseMass));
    }
  }

  private resolvePenetration(
    a: PhysicsObject,
    b: PhysicsObject,
    manifold: CollisionManifold
  ): void {
    const percent = 0.2; // Penetration percentage to correct
    const slop = 0.01; // Allowable penetration
    const correction = manifold.normal.multiply(
      Math.max(manifold.penetration - slop, 0) /
      (a.inverseMass + b.inverseMass) * percent
    );

    if (!a.isStatic) {
      a.position.subtract(correction.multiply(a.inverseMass));
    }
    if (!b.isStatic) {
      b.position.add(correction.multiply(b.inverseMass));
    }
  }
}
```

## Pattern: Physics Islands and Sleeping

### Good Example: Island Management
```typescript
// ✅ Optimize by grouping and sleeping inactive objects
class PhysicsIslandManager {
  private islands: Set<PhysicsIsland> = new Set();
  private sleepThreshold = 0.1; // Velocity threshold for sleeping
  private sleepTime = 1.0; // Time before object sleeps

  update(deltaTime: number): void {
    // Build islands of connected objects
    this.buildIslands();

    for (const island of this.islands) {
      if (island.isAwake) {
        this.updateIsland(island, deltaTime);

        // Check if island can sleep
        if (island.shouldSleep(this.sleepThreshold, this.sleepTime)) {
          island.sleep();
        }
      }
    }
  }

  private buildIslands(): void {
    this.islands.clear();
    const visited = new Set<PhysicsObject>();

    for (const obj of this.allObjects) {
      if (!visited.has(obj)) {
        const island = new PhysicsIsland();
        this.buildIslandDFS(obj, island, visited);

        if (island.objects.size > 0) {
          this.islands.add(island);
        }
      }
    }
  }

  private buildIslandDFS(
    obj: PhysicsObject,
    island: PhysicsIsland,
    visited: Set<PhysicsObject>
  ): void {
    if (visited.has(obj)) return;

    visited.add(obj);
    island.add(obj);

    // Add connected objects (via constraints or contacts)
    for (const contact of obj.contacts) {
      this.buildIslandDFS(contact.other, island, visited);
    }

    for (const constraint of obj.constraints) {
      this.buildIslandDFS(constraint.other, island, visited);
    }
  }
}

class PhysicsIsland {
  objects: Set<PhysicsObject> = new Set();
  isAwake: boolean = true;
  private sleepTimer: number = 0;
  private maxKineticEnergy: number = 0;

  add(obj: PhysicsObject): void {
    this.objects.add(obj);

    if (obj.isAwake) {
      this.wake();
    }
  }

  wake(): void {
    this.isAwake = true;
    this.sleepTimer = 0;

    for (const obj of this.objects) {
      obj.wake();
    }
  }

  sleep(): void {
    this.isAwake = false;

    for (const obj of this.objects) {
      obj.sleep();
      obj.velocity.set(0, 0);
      obj.angularVelocity = 0;
    }
  }

  shouldSleep(threshold: number, requiredTime: number): boolean {
    let totalEnergy = 0;

    for (const obj of this.objects) {
      const kineticEnergy =
        0.5 * obj.mass * obj.velocity.lengthSquared() +
        0.5 * obj.inertia * obj.angularVelocity * obj.angularVelocity;

      totalEnergy += kineticEnergy;
    }

    if (totalEnergy < threshold) {
      this.sleepTimer += deltaTime;
      return this.sleepTimer >= requiredTime;
    }

    this.sleepTimer = 0;
    return false;
  }
}
```

## Performance Optimization Strategies

### Good Example: LOD for Physics
```typescript
// ✅ Different physics quality based on importance
class LODPhysicsSystem {
  private detailLevels = {
    high: { substeps: 4, ccdEnabled: true, frictionIterations: 8 },
    medium: { substeps: 2, ccdEnabled: true, frictionIterations: 4 },
    low: { substeps: 1, ccdEnabled: false, frictionIterations: 2 },
    minimal: { substeps: 1, ccdEnabled: false, frictionIterations: 0 }
  };

  updateObject(obj: PhysicsObject, deltaTime: number): void {
    const lod = this.calculateLOD(obj);
    const settings = this.detailLevels[lod];

    const substepDelta = deltaTime / settings.substeps;

    for (let i = 0; i < settings.substeps; i++) {
      if (settings.ccdEnabled && obj.velocity.length() > this.ccdThreshold) {
        this.continuousUpdate(obj, substepDelta);
      } else {
        this.discreteUpdate(obj, substepDelta);
      }
    }
  }

  private calculateLOD(obj: PhysicsObject): 'high' | 'medium' | 'low' | 'minimal' {
    // Distance from camera
    const distanceToCamera = obj.position.distanceTo(camera.position);

    // Importance factors
    const isPlayerInteracting = obj.tags.has('player_interactive');
    const isVisible = frustum.containsPoint(obj.position);
    const isMovingFast = obj.velocity.length() > 50;

    if (isPlayerInteracting || distanceToCamera < 10) {
      return 'high';
    } else if (isVisible && distanceToCamera < 50) {
      return isMovingFast ? 'high' : 'medium';
    } else if (isVisible && distanceToCamera < 100) {
      return 'low';
    } else {
      return 'minimal';
    }
  }
}
```

## Best Practices

1. **Use fixed timestep for physics** - Variable timestep causes instability
2. **Implement broad phase optimization** - Never check all pairs
3. **Sleep inactive objects** - Don't simulate static scenes
4. **Use appropriate integration method** - Verlet for constraints, RK4 for precision
5. **Profile bottlenecks** - Collision detection is usually the culprit
6. **Cache collision data** - Reuse manifolds between frames
7. **Separate static and dynamic objects** - Different optimization strategies
8. **Use LOD for distant physics** - Reduce quality where unnoticeable
9. **Batch similar operations** - SIMD optimization opportunities
10. **Provide deterministic option** - Required for networked games

Remember: Game physics doesn't need to be realistic, it needs to be fun and stable.