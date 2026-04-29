## UI 和CLI支持更新指定的plugins到指定的版本
但是会进行swcd校验，目前有些工程会报ERROR导致被update的plugin移除；
解决方式：实测，通过先移除，但不删除data xdm file，然后再次执行import方式可以解决；
注意：此时需要为当前plugin重设output dir等信息；
后续如果搞自动update，可以用该方式；
具体可以查EB tresos user guide