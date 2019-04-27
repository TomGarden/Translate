## android.support.v4.widget.SwipeRefreshLayout

SwipeRefreshLayout 应该应用于任何用户可以通过下滑手势触发内容更新的地方。
持有它的 Activity 应该增加一个 OnRefreshListener 用于接收刷新手势生效的通知。
SwipeRefreshLayout 将会在每次刷新手势完成后通知 OnRefreshListener ；
对内容实质性的刷新动作是否启动由监听器决定。
如果监听器认为不应该刷新，它应该调用 `setRefreshing(false)` 去取消视觉效果。
如果一个 activity 仅仅希望展示过程动画，应该调用 `setRefreshing(true)` 。
要取消手势并和执过程动画应该调用 `setEnable(false)` 。

这个布局应该作为可刷新布局的父布局被使用(同时 SwipeRefreshLayout 仅有一个子布局)。
这个 view 被设置为手势的消耗者，并且强制消耗它所在范围内的手势事件。
SwiperFreshLayout 不提供辅助功能事件；相反，必须提供菜单项，以允许在使用此笔势的任何位置刷新内容。