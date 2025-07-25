# ArcFileWritten: Windows

# Description

Platforms: Windows, macOS, Linux

Emitted when a process is done writing a ARC file.

Fields: Windows


| Field  | Description  |
| --- | --- |
| ContextTimeStamp  | System time of event creation.  |
| ContextProcessId  | UPID of process originating this event.  |
| ContextThreadId  | UTID of thread originating this event  |
| TreeId  | If this event is part of a detection tree, the tree ID it is part of.  |
| FileObject  |  |
| MajorFunction  |  |
| MinorFunction  |  |
| IrpFlags  |  |
| OperationFlags  |  |
| Size  |  |
| DiskParentDeviceInstanceId  |  |
| FileIdentifier  |  |
| TargetFileName  | The resulting file name that was downloaded  |
| IsOnNetwork  | Set to true if the relevant file listed in the event is on a network drive. False otherwise.  |
| IsOnRemovableDisk  | If true, it means this file was located on a removable disk.  |
|  | Values: |
| AuthenticationId  | INVALID_LUID (0)<br>NETWORK_SERVICE (996)<br>LOCAL_SERVICE (997)<br>SYSTEM (999)<br>RESERVED_LUID_MAX (1000)  |
| TokenType  | Values:<br>INVALID_TOKEN (0)<br>PRIMARY_TOKEN (1)<br>IMPERSONATION_TOKEN (2)  |
| UserName  |  |
| FileEcpBitmask  | Values:<br>ECP_SRV_OPEN (0x00000001)<br>ECP_CS_SENSOR_OPEN (0x00000002)<br>ECP_CS_SENSOR_FILE_CHARACTERISTICS (0x00000004)  |
| TemporaryFileName  |  |
| FileWrittenFlags  | Values:<br>HASH_IS_VALID (0x00000000)<br>HASH_FAILED (0x00000001)<br>HASH_ABORTED_TOO_LARGE (0x00000002)<br>PAGING_WRITE (0x00000004)  |
| FileOperatorSid  |  |
| FileCategory  | Values:<br>OTHER (0)<br>ARCHIVES (1)<br>OFFICE_DOCUMENTS (2)<br>MULTIMEDIA_FILES (3)<br>DESIGN_FILES (4)<br>SOURCE_CODE (5)<br>EXECUTABLE_FILES (6)<br>VIRTUAL_MACHINE_FILES (7)<br>EMAIL_FILES (8)<br>DATA_AND_LOGS (9)<br>ENCRYPTED (10)  |
| VolumeSessionUUID  |  |
| FileSourcePath  |  |
| MipInformation  |  |
| SHA256HashData  | The SHA256 hash of a file. In most cases, the hash of the file referred to by the ImageFileName field.<br>Values:<br>STATIC_SHA256_DOPPELGANGING (0x56f3097c4d5bf4c7cﬀef168ee732e1c78f2ee62bc1c1ba61c219226bef619f8)STATIC_SHA256_SYSTEM (0x03312a19baa7ab137c09127c6feb58c05216a7880d3c9e6ae54a8bcda460f92a)  |
| ContextBaseFileName  |  |


