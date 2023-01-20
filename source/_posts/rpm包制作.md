---
title: rpm包制作
date: 2014-08-17 11:30:55
tags:
    - LINUX
---

1. 设置打包目录  
a) 查看当前设置：
```shell
rpmbuild --showrc | grep topdir
```
b) 设置目录：
```shell
vim ~/.rpmmacros
%_topdir /root/src/rpmbuild

> mkdir rpmbuild
> cd ./rpmbuild
> mkdir -pv {BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}
```

2. 拷贝源代码source 目录
```shell
>cp  /root/src/tengine-2.0.3.tar.gz SOURCES/
```

3. 建立spec文件
```shell
> cd SPECS
> vim tengine-2.0.3.spec
Name: tengine
Version: 2.0.3
Release: 1%{?dist}
Summary: tengine from Taobao

Group: System Environment/Daemons
License: GPLv2
URL: http://blog.zhangwenjin.com
Source0: %{name}-%{version}.tar.gz
BuildRoot: %(mktemp -ud %{_tmppath}/%{name}-%{version}-%{release}-XXXXXX)

BuildRequires: gcc,make
Requires: openssl,pcre,pcre-devel,lua,lua-devel

%description
It is a Nginx from Taobao.

%prep
%setup -q


%build
./configure --with-http_lua_module --with-pcre --user=nginx --group=nginx
make %{?_smp_mflags}


%install
rm -rf %{buildroot}
make install DESTDIR=%{buildroot}


%clean
rm -rf %{buildroot}


%files
%defattr(-,root,root,-)
%doc
```
4. rpmbuild 打包

rpmbuild 
- -ba 既生成src.rpm又生成二进制rpm 
- -bs 只生成src的rpm 
- -bb 只生二进制的rpm 
- -bp 执行到pre 
- -bc 执行到 build段 
- -bi 执行install段 
- -bl 检测有文件没包含

```shell
> rpmbuild -bb tengine-2.0.3.spec
```
