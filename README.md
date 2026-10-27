# CPU-Stress-Tests
Ryzen 5 5900X stress tests of core 2 on processors 2, and 8

```bash
sudo apt update
sudo apt install stress-ng lm-sensors
```

```bash
for cpu in 0 2 8; do
    echo "===== TESTING CPU $cpu ====="

    taskset -c "$cpu" stress-ng \
        --cpu 1 \
        --cpu-method all \
        --verify \
        --timeout 10m \
        --metrics-brief \
        2>&1 | tee "$HOME/stress-cpu${cpu}.log"
done
```

## On a separate terminal

```bash
watch -n 2 sensors
```

## Timestamps for temp signatures

```bash
while true; do date --iso-8601=seconds; sensors; sleep 5; done \
  | tee "$HOME/cpu-core2-diagnostics/temperatures-live.log"
```

### Optional tests on individual CPUs

```bash
cd "$HOME/cpu-core2-diagnostics"
set -o pipefail

# Control: logical CPU 0 (physical core 0)
taskset -c 0 stress-ng --cpu 1 --cpu-method all --verify \
  --timeout 10m --metrics-brief 2>&1 | tee stress-cpu0.log

# Suspect: logical CPU 2 (physical core 2)
taskset -c 2 stress-ng --cpu 1 --cpu-method all --verify \
  --timeout 10m --metrics-brief 2>&1 | tee stress-cpu2.log

# Suspect: logical CPU 8 (SMT sibling of CPU 2)
taskset -c 8 stress-ng --cpu 1 --cpu-method all --verify \
  --timeout 10m --metrics-brief 2>&1 | tee stress-cpu8.log
```

### Optional stress both threads of physical core 2 simultaneously

```bash
cd "$HOME/cpu-core2-diagnostics"

taskset -c 2,8 stress-ng --cpu 2 --cpu-method all --verify \
  --timeout 10m --metrics-brief 2>&1 | tee stress-core2-both-threads.log
```

### For comparison, we can test both threads of physical core 0:

```bash
taskset -c 0,6 stress-ng --cpu 2 --cpu-method all --verify \
  --timeout 10m --metrics-brief 2>&1 | tee stress-core0-both-threads.log
```

## Collect tests results evidences:

```bash
journalctl -k --since "$TEST_START" --no-pager \
  | tee "$HOME/cpu-core2-diagnostics/kernel-during-tests.log"
```
