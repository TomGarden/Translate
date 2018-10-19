## 0x00、 SharedPreferences.Editor

> public abstract boolean commit ()

提交对手选项的修改到当前 Editor 对应的 SharedPreference 。
这将最细粒度地执行对其对应的 SharedPreference 的修改替换或任何其他操作。

注意：当两个 Editor 同时修改同一个首选项文件的时候，以后 commit 的为准。

如果你打算在主线程中调用这个方法并且不关心它的返回值，建议你调用 apply() 替代。


> public abstract void apply ()

提交对手选项的修改到当前 Editor 对应的 SharedPreference 。
这将最细粒度地执行对其对应的 SharedPreference 的修改替换或任何其他操作。

注意：当两个 Editor 同时修改同一个首选项文件的时候，以后 apply 的为准。

和 commit 的同步操作不同，调用 apply 提交修改内容后，修改内容会被存储到内存中，在未来某个不确定的时间内异步本地化到磁盘。
如果在 apply 尚未完成之前 SharedPreference 被 commit 了其他内容，commit 会被阻塞直到 apply 完成。

由于 SharedPreference 在进程中是单例，所以任何时候将 apply 替换为 commite 都是安全的。