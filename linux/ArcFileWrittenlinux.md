# ArcFileWritten: Linux

# Description

Platforms: Windows, macOS, Linux

Emitted when a process is done writing a ARC file.



Fields: Linux


| Field  | Description  |
| --- | --- |
| ContextTimeStamp  | System time of event creation.  |
| ContextProcessId  | UPID of process originating this event.  |
| ContextThreadId  | UTID of thread originating this event  |
| TreeId  | If this event is part of a detection tree, the tree ID it is part of.  |
| FileIdentifier  |  |
| TargetFileName  | The resulting file name that was downloaded  |
| IsOnRemovableDisk  | If true, it means this file was located on a removable disk.  |
| UserName  |  |
| Size  |  |
| DiskParentDeviceInstanceId  |  |
| FileCategory  | Values:<br>OTHER (0)<br>ARCHIVES (1)<br>OFFICE_DOCUMENTS (2) MULTIMEDIA_FILES (3)<br>DESIGN_FILES (4)<br>SOURCE_CODE (5)<br>EXECUTABLE_FILES (6)<br>VIRTUAL_MACHINE_FILES (7)<br>EMAIL_FILES (8)<br>DATA_AND_LOGS (9)<br>ENCRYPTED (10)  |
| VolumeSessionUUID  |  |
| FileSourcePath  |  |
| MipInformation  |  |
| SHA256HashData  | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field.<br>Values:<br>STATIC_SHA256_DOPPELGANGING (0x56f3097c4d5bf4c7cﬀef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8)<br>STATIC_SHA256_SYSTEM (0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a)  |
| MountNamespaceUniqueId  |  |
| RawProcessId  | The operating system’s internal PID. For matching, use the UPID fields which guarantee a unique process identifier  |
| RUID  | The real UID for a unix style process.  |
| RGID  | The real GID for a unix style process.  |
| FsMagic  |  |
| DeviceMountCounter  |  |
| DeviceId  |  |
| FileSerialNumber  |  |
| ChangeTime  | Unix change time, indicating last status change or file modification. NOTE: This is the only timestamp which is not modifiable,<br>unlike the other file timestamps.  |
| ContextBaseFileName  |  |


