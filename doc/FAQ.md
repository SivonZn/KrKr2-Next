## 安装glib失败
### 在Wndows为Android进行跨平台编译时遇到vcpkg无法安装glib时
#### 原因
- 是因为 meson 生成的 install.dat 的路径不正常如
```
D:/source/vcpkg/packages/glib_arm64-android/debug/share/gdb/auto-load/./D:/source/vcpkg/packages/glib_arm64-android/debug/lib
```
- 需要修改`meson`的`minstall.py`

#### 具体步骤
1. 根据`vcpkg的dbg err 信息`锁定是哪一个'meson'如果没有更改`glib`版本那么应该是`1.6.1`
2. 进入`meson`目录,在`VCPKG_ROOT/downloads/tools/meson-1.6.1-哈希值/mesonbuild`,我的路径是`VCPKG_ROOT\vcpkg\downloads\tools\meson-1.6.1-6779de\mesonbuild` 
3. 打开`minstall.py`
4. 替换`install_data`
```py
def install_data(self, d: InstallData, dm: DirMaker, destdir: str, fullprefix: str) -> None:
        

        for i in d.data:
            if not self.should_install(i):
                continue
            if "/./" in i.install_path:
                i.install_path = i.install_path.split('.')[0]+i.install_path.split('\\')[-1]
            fullfilename = i.path
            outfilename = get_destdir_path(destdir, fullprefix, i.install_path)
            
            outdir = os.path.dirname(outfilename)
            try:
                if self.do_copyfile(fullfilename, outfilename, makedirs=(dm, outdir), follow_symlinks=i.follow_symlinks):
                    self.did_install_something = True
            except Exception as e:
                print(f"Error installing {fullfilename} to {outfilename}: {e}")
            self.set_mode(outfilename, i.install_mode, d.install_umask)
```

## 编译时部分依赖库下载错误
例如
```bash
......
Installing 1/27 fribidi:x64-osx@1.0.16...
fribidi:x64-osx@1.0.16 package ABI: c27e7d87891f0ee957f434c25d373aa2e90cae9350c7b23e2507c157d2c121b5
Building fribidi:x64-osx@1.0.16...
/Users/sivon/Downloads/KrKr2-Next/.devtools/vcpkg/triplets/community/x64-osx.cmake: info: loaded community triplet from here. Community triplets are not built in the curated registry and are thus less likely to succeed.
-- Found Python version '3.14.2 at /usr/local/bin/python3'
-- Using meson: /Users/sivon/Downloads/KrKr2-Next/.devtools/vcpkg/downloads/tools/meson-1.9.0-7310ba/meson.py
Downloading https://github.com/fribidi/fribidi/archive/v1.0.16.tar.gz -> fribidi-fribidi-v1.0.16.tar.gz
error: curl operation failed with error code 18 (Transferred a partial file).
......
```
#### 解决方法：
参照`Downloading https://github.com/fribidi/fribidi/archive/v1.0.16.tar.gz -> fribidi-fribidi-v1.0.16.tar.gz`这句报错，可以看出`vcpkg`在下载`fribidi`并将其重命名为`fribidi-fribidi-v1.0.16.tar.gz`，所以我们可以：
1. 手动下载依赖库，这里以`fribidi`为例：https://github.com/fribidi/fribidi/archive/refs/tags/v1.0.16.tar.gz
2. 将其重命名为`fribidi-fribidi-v1.0.16.tar.gz`
3. 手动将压缩包放入`.devtools/vcpak/downloads`目录内