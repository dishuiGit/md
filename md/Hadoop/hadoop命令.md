[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

### [ hadoop fs][anchor-cur]

__Usage: hadoop fs [generic options]__

+ [-appendToFile <localsrc> ... <dst>]
+ [-cat [-ignoreCrc] <src> ...]
+ [-checksum <src> ...]
+ [-chgrp [-R] GROUP PATH...]
+ [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
+ [-chown [-R] [OWNER][:[GROUP]] PATH...]
+ [-copyFromLocal [-f] [-p] <localsrc> ... <dst>]
+ [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
+ [-count [-q] <path> ...]
+ [-cp [-f] [-p] <src> ... <dst>]
+ [-createSnapshot <snapshotDir> [<snapshotName>]]
+ [-deleteSnapshot <snapshotDir> <snapshotName>]
+ [-df [-h] [<path> ...]]
+ [-du [-s] [-h] <path> ...]
+ [-expunge]
+ [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
+ [-getfacl [-R] <path>]
+ [-getmerge [-nl] <src> <localdst>]
+ [-help [cmd ...]]
+ [-ls [-d] [-h] [-R] [<path> ...]]
+ [-mkdir [-p] <path> ...]
+ [-moveFromLocal <localsrc> ... <dst>]
+ [-moveToLocal <src> <localdst>]
+ [-mv <src> ... <dst>]
+ [-put [-f] [-p] <localsrc> ... <dst>]
+ [-renameSnapshot <snapshotDir> <oldName> <newName>]
+ [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
+ [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
+ [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
+ [-setrep [-R] [-w] <rep> <path> ...]
+ [-stat [format] <path> ...]
+ [-tail [-f] <file>]
+ [-test -[defsz] <path>]
+ [-text [-ignoreCrc] <src> ...]
+ [-touchz <path> ...]
+ [-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|jobtracker:port>    specify a job tracker
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
