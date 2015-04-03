# Overall Conventions

Items here apply to all interface classes. Note that these conventions hold within the OSVR system. Game engine integrations may (and should) adapt to the engine's native conventions.

## Units

- Distance/length: meters
- Orientation: unit (normalized) quaternions in a right-handed system
- Time (for use in derivatives): seconds, unless otherwise noted
- Linear velocity: m/s
- Linear acceleration: m/s^2
- Angular velocity: an incremental unit (normalized) quaternion and a time interval.
- Angular acceleration: an incremental unit (normalized) quaternion and a time interval.

## Coordinate Systems
The global OSVR coordinate system is right-handed, with _x_ to the right and _y_ up. When depicted with only color as a label, axes match RGB-XYZ respectively.

![World Coordinate System](world-axes.png)
