Scheduler
===

The idea is to store a stream of messages on a microcontroller which is replayed later (either once or repeated). A task is created by sending a create_task message. The time-to-run is initialized with 0 (which means the task is not yet ready to run). After filling up the taskdata with messages (using add_to_task command messages) a final schedule_task request is sent, that sets the time-to-run (in milliseconds after 'now'). If a task itself contains delay_task or schedule_task-messages these cause the execution of the task to pause and resume after the amount of time given in such message has elapsed. If the last message in a task is a delay_task message the task is scheduled for reexecution after the amount of time specified. If there's no delay_task message at the end of the task (so the time-to-run is not updated during the run) the task gets deleted after execution.

Added in version 2.4.0.


### Example files: 
 * OneWire is include by default in [ConfigurableFirmata.ino](https://github.com/firmata/ConfigurableFirmata/blob/master/examples/ConfigurableFirmata/ConfigurableFirmata.ino). 
 * [Example implementation](https://github.com/firmata/ConfigurableFirmata/blob/master/src/FirmataScheduler.cpp) as a configurable Firmata feature class.


### Compatible host implementations
* [ConfigurableFirmata](https://github.com/firmata/ConfigurableFirmata)


### Compatible client librairies
* [perl-firmata](https://github.com/ntruchsess/perl-firmata)


### Protocol details

Scheduler CREATE_TASK request
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  create_task command  (0x00)
3  task id              (0-127)
4  length LSB           (bit 0-6)
5  length MSB           (bit 7-13)
6  END_SYSEX            (0xF7)
```

Scheduler DELETE_TASK request
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  delete_task command  (0x01)
3  task id              (0-127)
4  END_SYSEX            (0xF7)
```

Scheduler ADD_TO_TASK request
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  add_to_task command  (0x02)
3  task id              (0-127)
4  taskdata bit 0-6     [optional] task bytes encoded using 8 times 7 bit
                                   for 7 bytes of 8 bit
5  taskdata bit 7-13    [optional]
6  taskdata bit 14-20   [optional]
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX          (0xF7)
```

Scheduler DELAY_TASK request
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  delay_task command   (0x03)
3  time_ms bit 0-6      time_ms is of type long, requires 32 bit.
4  time_ms bit 7-13
5  time_ms bit 14-20
6  time_ms bit 21-27
7  time_ms bit 28-31
8  END_SYSEX            (0xF7)
```

Scheduler SCHEDULE_TASK request
```
0  START_SYSEX              (0xF0)
1  Scheduler Command        (0x7B)
2  schedule_task command    (0x04)
3  task id                  (0-127)
4  time_ms bit 0-6          time_ms is of type long, requires 32 bit.
5  time_ms bit 7-13
6  time_ms bit 14-20
7  time_ms bit 21-27
8  time_ms bit 28-31
9  END_SYSEX                (0xF7)
```

Scheduler QUERY_ALL_TASKS request
```
0  START_SYSEX              (0xF0)
1  Scheduler Command        (0x7B)
2  query_all_tasks command  (0x05)
3  END_SYSEX                (0xF7)
```

Scheduler QUERY_ALL_TASKS reply
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  query_all_tasks Reply Command (0x09)
3  taskid_1             (0-127) [optional]
4  taskid_2             (0-127) [optional]
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX (0xF7)
```

Scheduler QUERY_TASK request
```
0  START_SYSEX              (0xF0)
1  Scheduler Command        (0x7B)
2  query_task command       (0x06)
3  task id                  (0-127)
4  END_SYSEX                (0xF7)
```

Scheduler QUERY_TASK reply
```
0  START_SYSEX          (0xF0)
1  Scheduler Command    (0x7B)
2  query_task Reply Commandc (0x0A)
3  task id              (0-127)
4  time_ms bit 0-6
5  time_ms bit 7-13
6  time_ms bit 14-20
7  time_ms bit 21-27
8  time_ms bit 28-31 | (length bit 0-2) << 4
9  length bit 3-9
10 length bit 10-15 | (position bit 0) << 7
11 position bit 1-7
12 position bit 8-14
13 position bit 15 | taskdata bit 0-5 << 1 [taskdata is optional]
14 taskdata bit 6-12  [optional]
15 taskdata bit 13-19 [optional]
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX          (0xF7)
```

Scheduler RESET request
```
0  START_SYSEX              (0xF0)
1  Scheduler Command        (0x7B)
2  scheduler reset command  (0x07)
3  END_SYSEX                (0xF7)
```

Scheduler ERROR_FIRMATA_TASK reply
```
0  START_SYSEX              (0xF0)
1  Scheduler Command        (0x7B)
2  error_task Reply Command (0x08)
3  task id                  (0-127)
4  time_ms bit 0-6
5  time_ms bit 7-13
6  time_ms bit 14-20
7  time_ms bit 21-27
8  time_ms bit 28-31 | (length bit 0-2) << 4
9  length bit 3-9
10 length bit 10-15 | (position bit 0) << 7
11 position bit 1-7
12 position bit 8-14
13 position bit 15 | taskdata bit 0-5 << 1 [taskdata is optional]
14 taskdata bit 6-12  [optional]
15 taskdata bit 13-19 [optional]
n  ... as many bytes as needed (don't exceed MAX_DATA_BYTES though)
n+1  END_SYSEX              (0xF7)
```
