# 03-lab-agent-daemon.md

## Specification: Lab Node Agent
This compiled daemon runs directly on the OS of the lab PCs. It handles the local delta compression, offline buffering, firewall manipulation, and ephemeral account provisioning.

## Implementation Tasks

- [ ] **Task 1: Implement Ephemeral Account Provisioning**
  - **Action**: Write OS-specific system calls (or shell execs) to create a temporary, locked-down local user profile upon receiving a "Start Exam" broadcast, and purge the profile directory completely on "End Exam".
  - **Verification**: Write a test that provisions a user, writes a file, and purges it, asserting the OS is clean afterward. `go test -run TestEphemeralProfile`
  - **Done**: The environment orchestration logic can dynamically create and destroy isolated user spaces.

- [ ] **Task 2: File Watcher & Delta Compression Engine**
  - **Action**: Implement `fsnotify` to detect file saves, and a 30-second periodic timer to catch unsaved IDE buffers. When a change is detected, run a diffing algorithm locally against the previous state, generate a patch payload, and queue it.
  - **Verification**: Write a unit test that modifies a file 5 times and ensures only the 5 calculated diffs are generated, not the whole file. `go test -run TestDeltaSync`
  - **Done**: The Agent successfully generates lightweight patches instead of full files.

- [ ] **Task 3: Implement Offline Buffering (Fault Tolerance)**
  - **Action**: If the WebSocket connection drops, divert all outgoing delta patches to a local, encrypted SQLite queue. Once the WebSocket reconnects, flush the queue sequentially to the Broker before resuming live sync.
  - **Verification**: Write an integration test where the network is disabled, files are modified, and network is restored. `go test -run TestFaultTolerance`
  - **Done**: No file changes are lost during simulated network outages.

- [ ] **Task 4: Implement Programmatic Firewall Lockdown**
  - **Action**: Write a system adapter that interfaces with the host OS firewall (`iptables`/`netsh`). On exam start, apply rules that drop all outgoing WAN traffic but whitelist the Broker's local IP address.
  - **Verification**: Write a mock test verifying the correct shell commands are generated. `go test -run TestFirewall`
  - **Done**: Agent successfully alters the host's networking rules to isolate the machine.