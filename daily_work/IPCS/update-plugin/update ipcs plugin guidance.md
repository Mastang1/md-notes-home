## 1. 修改项
_**1.修改IPCS plugin文件夹，目前手动替换，后续可以搞一个脚本自动化update
2.修改driver中version信息
3.修改ipcs-test/full_sys_demo/IPCS examples/ut_configuration等4个地方的ipcs data xdm文件，可以用CLI自动update，但是swcd校验出错，所以不可用；但是可以删除旧版本的的plugin，但是保留data.xdm,加入新版本时候点击“Load existing file”实现自动更新，但是保留修原始data.xdm
4.前序修改、验证通过后，将新开发的plugin加入到/data/eb_remote/...及PC-41(CI及自动生成用，在本地)，保持多版本共存
5.执行ipcs工程、mt、ut三个工程push操作
**_


