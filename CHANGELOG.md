# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Control over client visibility of entities.

### Changed

- Rename `ClientsInfo` into `ClientCache`.
- Rename `ClientInfo` into `ClientState`.
- Allow calling `dont_replicate::<T>` with the insertion of `T` instead of after `Replication` insertion.
- `dont_replicate::<T>` can panic only in `debug_assertions` is enabled.

## [0.20.0] - 2024-01-13

### Fixed

- Don't panic when handling client acks if the ack references a despawned entity.

### Changed

- API for custom server messages now uses `server_event::serialize_with`  and `server_event::deserialize_with`. For more details see the example in the docs.
- Speedup serialization for multiple clients by reusing already serialized components and entities.
- Hide extra functionality from `ServerEventQueue`.
- Move server event reset system to new set `ClientSet::ResetEvents` in `PreUpdate`.
- Make `NetworkChannels` channel-creation methods public (`create_client_channel()` and `create_server_channel()`).
- Implement `Eq` and `PartialEq` on `EventType`.

### Removed

- `LastChangeTick` resource, `ClientsInfo` should be used instead.

## [0.19.0] - 2024-01-07

### Added

- `renet_serde` feature which reexports `serde` feature from `bevy_renet`.
- `ClientSet::Reset` which can be disabled by external users.
- `ServerEntityMap::remove_by_client()` for manual client cleanup.
- `BufferedUpdates`, `ServerEntityTicks` to public API.

### Changed

- Move the client reset system to `PreUpdate` to let clients react more promptly to resets.
- Replace `Ignored<T>` with `CommandDontReplicateExt::dont_replicate`.
- `Replication` entities with no replicated components will now be spawned on the client anyway.

## [0.18.2] - 2023-12-27

### Fixed

- Fix missing removals and despawns caused by events cleanup.

## [0.18.1] - 2023-12-21

### Fixed

- Fix crash caused by registering the same type for client and server events.
- Fix replication for entities when `Replication` component is added after spawn.

### Changed

- Cache replicated archetypes for faster iteration.

## [0.18.0] - 2023-12-19

### Fixed

- Fix missing reset of `RepliconTick` on server disconnect.
- Fix replication of removals that happened after replication on the same frame.

### Changed

- Send all component mappings, inserts, removals and despawns over reliable channel in form of deltas and component updates over unreliable channel packed by packet size. This significantly reduces the possibility of packet loss.
- Replace `REPLICATION_CHANNEL_ID` with `ReplicationChannel` enum. The previous constant corresponded to the unreliable channel.
- Server events use tick with the last change instead of waiting for replication message without changes.
- Include despawns before removals to optimize space for case where despawns are presents and removals aren't.
- `TickPolicy::EveryFrame` and `TickPolicy::MaxTickRate` now increment tick only if `RenetServer` exists.
- `ServerSet::Send` now always runs. Replication sending system still runs on `RepliconTick` change.
- `ClientMapping` no longer contains `tick` field.
- Use `EntityHashMap` instead of `HashMap` with entities as keys.
- Use `Cursor<&[u8]>` instead of `Cursor<Bytes>`.
- Replace `LastRepliconTick` with `RepliconTick` on client.
- Move `ClientMapper` and `ServerEntityMap` to `client_mapper` submodule.
- Rename `replicate_into_scene` into `replicate_into` and move it to `scene` module.
- Derive `Debug` for `Replication` and `Ignored<T>`.

### Removed

- `AckedTicks` resource.
- `TicksMap` resource.

## [0.17.0] - 2023-11-13

### Added

- Tracing for replication messages.
- `Debug` derive for `LastRepliconTick`.

### Changed

- Update to Bevy 0.12.

## [0.16.0] - 2023-10-30

### Added

- API to configure max channel usage bytes in `NetworkChannels`.

### Changed

- Rename `SendPolicy` into `EventType`.
- Rename `NetworkChannels::server_channels` into `NetworkChannels::get_server_configs`.
- Rename `NetworkChannels::client_channels` into `NetworkChannels::get_client_configs`.

## [0.15.1] - 2023-10-22

### Changed

- Register `Replication` type and add `#[reflect(Component)]`.

## [0.15.0] - 2023-10-21

### Added

- `network_event::server_event::send` helper for server events in custom sending functions.
- Optional `ClientDiagnosticsPlugin`, which writes diagnostics every second.
- `Reflect` derive for `Replication`.

### Changed

- Optimize despawn tracking.
- Hide `id` field in `EventChannel` and add `Clone` and `Copy` impls for it.
- Remove special functions for reflect events and advise users to write them manually instead. Reflect events are easier now because sometimes you can directly use reflect serializers from Bevy instead of manually writing serde traits.
- Do no trigger server events before world update arrival.

### Removed

- `Debug` requirement for events.

## [0.14.0] - 2023-10-05

### Added

- The ability to pre-spawn entities on client.

### Changed

- Rename `NetworkEntityMap` to `ServerEntityMap`.

## [0.13.0] - 2023-10-04

### Added

- The ability to set custom despawn and component removal functions.
- `TickPolicy::EveryFrame` to update `RepliconTick` every frame.

### Fixed

- Fix the entire world was always sent instead of changes.
- Fix crash with several entities spawned and updated.

### Changed

- Use more compact varint encoding for entities.
- Now all replication functions accept `RepliconTick`.
- Rename `NetworkTick` into `RepliconTick` and move it into `server` module.
- Rename `LastTick` into `LastRepliconTick`.

### Removed

- `derive_more` dependency.

## [0.12.0] - 2023-10-01

### Added

- High-level API to extract replicated entities into `DynamicScene`.

### Changed

- Hide `ReplicationRules` from public API.
- Move logic related to replication rules to `replicon_core::replication_rules` module.

## [0.11.0] - 2023-09-25

### Changed

- Serialize all components and events using varint.
- Serialize entities in optimal way by writing its index and generation as separate varints.
- Hide `ReplicationId`, `ReplicationInfo` and related methods from `ReplicationRules` from public API.
- Rename `ReplicationRules::replication_id` into `ReplicationRules::replication_marker_id`.
- Use serialization buffer cache per client for replication.
- Correctly handle old values on packet reordering.
- Bevy's `Tick` was replaced with dedicated type `NetworkTick` that increments on server update, so it can be used to provide information about time. `AckedTick` was replaced with `ServerTicks` that also contains mappings from `NetworkTick` to Bevy's `Tick` and current `NetworkTick`.
- Functions in `AppReplicationExt::replicate_with` now accept bytes cursor for memory reuse and return serialization errors.
- Rename `ReplicationCore` into `RepliconCore` with its module for clarity.
- `MapNetworkEntities` now accepts generic `Mapper` and doesn't have error handling and deserialiation functions now accept `NetworkEntityMap`. This allowed us to lazily map entities on client without extra allocation.
- Make `LastTick` public.

## [0.10.0] - 2023-09-13

### Changed

- `MapEventEntities` was renamed into `MapNetworkEntities` and now used for both components and events. Built-in `MapEntities` trait from Bevy is not suited for network case for now.
- `AppReplicationExt::not_replicate_with` was replaced with component marker `Ignored<T>`.
- Reflection was replaced with plain serialization. Now components need to implement serde traits and no longer need any reflection. This reduced reduced message sizes a lot. Because of this mapped components now need to be registered with `AppReplicationExt::replicate_mapped`.
- Derive `Clone` and `Copy` for `Replication`.
- Make `ServerPlugin` fields private and add `ServerPlugin::new`.
- Make `AckedTicks` public.
- Make `NetworkEntityMap` public.

## [0.9.1] - 2023-08-05

### Fixed

- Fix event cleanup.

## [0.9.0] - 2023-08-01

### Added

- `ClientSet` now available from `prelude`.

### Changed

- Move `has_authority()` to `server` module.

## [0.8.0] - 2023-07-28

### Changed

- Put systems in `PreUpdate` and `PostUpdate` to avoid one frame delay.
- Reorganize `ServerSet` and move client-related systems to `ClientSet`.

## [0.7.1] - 2023-07-20

### Changed

- Re-export `transport` module from `bevy_renet`.

## [0.7.0] - 2023-07-20

### Changed

- Update to `bevy` 0.11.
- Mappable network events now need to implement `MapEventEntities` instead of `MapEntities`.

### Removed

- `ClientState` and `ServerState`, use conditions from `bevy_renet` and `resource_added()` / `resource_exists()` / `resource_removed()`.
- `ServerSet::Authority`, use `has_authority()` instead.

## [0.6.1] - 2023-07-09

### Changed

- Update `ParentSync` in `CoreSet::PostUpdate` to avoid one frame delay.

## [0.6.0] - 2023-07-08

### Added

- `SendPolicy` added to API of event-creation for user control of delivery guarantee (reliability and ordering).

### Changed

- `ParentSync` no longer accepts parent entity and just synchronizes hierarchy automatically if present.

## [0.5.0] - 2023-06-26

### Added

- `ServerSet::ReceiveEvent` and `ServerSet::SendEvent` for more fine-grained control of scheduling for event handling.

### Fixed

- Unspecified system ordering could cause tick acks to be ordered on the wrong side of world diff handling.
- Crash after adding events without `ServerPlugin` or `ClientPlugin`.

### Changed

- Update server to use `TickPolicy` instead of requiring a tick rate.

## [0.4.0] - 2023-05-26

### Changed

- Swap `registry` and `event` arguments in `BuildEventSerializer` for consistency with `ReflectSerializer`.
- Update to `bevy_renet` 0.0.12.

## [0.3.0] - 2023-04-15

### Added

- Support for sending events that contains `Box<dyn Reflect>` via custom serialization implementation.

### Changed

- Accept receiving system in `add_client_event_with` and sending system in `add_server_event_with`.
- Make `EventChannel<T>` public.

## [0.2.3] - 2023-04-09

### Fixed

- Panic that could occur when deleting `RenetServer` or `RenetClient` resources.

## [0.2.2] - 2023-04-05

### Fixed

- Do not panic if an entity was already despawned on client.

## [0.2.1] - 2023-04-02

### Fixed

- Incorrect last tick detection.

## [0.2.0] - 2023-04-01

### Changed

- Use `#[reflect(MapEntities)]` from Bevy 0.10.1 instead of custom `#[reflect(MapEntity)]`.

### Fixed

- Tick checks after overflow.

## [0.1.0] - 2023-03-28

Initial release after separation from [Project Harmonia](https://github.com/projectharmonia/project_harmonia).

[unreleased]: https://github.com/projectharmonia/bevy_replicon/compare/v0.20.0...HEAD
[0.20.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.19.0...v0.20.0
[0.19.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.18.2...v0.19.0
[0.18.2]: https://github.com/projectharmonia/bevy_replicon/compare/v0.18.1...v0.18.2
[0.18.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.18.0...v0.18.1
[0.18.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.17.0...v0.18.0
[0.17.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.16.0...v0.17.0
[0.16.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.15.1...v0.16.0
[0.15.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.15.0...v0.15.1
[0.15.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.14.0...v0.15.0
[0.14.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.13.0...v0.14.0
[0.13.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.12.0...v0.13.0
[0.12.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.11.0...v0.12.0
[0.11.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.10.0...v0.11.0
[0.10.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.9.1...v0.10.0
[0.9.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.9.0...v0.9.1
[0.9.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.8.0...v0.9.0
[0.8.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.7.1...v0.8.0
[0.7.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.7.0...v0.7.1
[0.7.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.6.1...v0.7.0
[0.6.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.6.0...v0.6.1
[0.6.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.5.0...v0.6.0
[0.5.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.4.0...v0.5.0
[0.4.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.3.0...v0.4.0
[0.3.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.2.3...v0.3.0
[0.2.3]: https://github.com/projectharmonia/bevy_replicon/compare/v0.2.2...v0.2.3
[0.2.2]: https://github.com/projectharmonia/bevy_replicon/compare/v0.2.1...v0.2.2
[0.2.1]: https://github.com/projectharmonia/bevy_replicon/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/projectharmonia/bevy_replicon/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/projectharmonia/bevy_replicon/releases/tag/v0.1.0
