---
id: ios-view
title: 视角同步
---

本文中的 API，都可以在 `WhiteRoom` 类中查看。本文示例代码中的 `room` 即为 `whiteRoom` 的实例。

White SDK 提供的白板是向四方无限延伸的。同时也允许用户通过鼠标滚轮、手势等方式移动白板。因此，即便是同一块白板的同一页，不同用户的屏幕上可能看到的内容是不一样的。为了满足，所有用户观看同一内容的需求，本文引入了「`主播模式`」这个概念。

## 主播视角

sdk 支持将房间内的某一个人设为主播（其他用户会自动变成 `观众模式`），该用户屏幕上看到的内容即是其他所有观众看到的内容。
当主播进行视角的放缩、移动时，其他人的屏幕也会自动进行放缩、移动等操作，来保证，可以观看到主播端所有的可见内容。

* 观众端显示的内容，多于主播端的情况

主播模式中，主播所看到的内容，会全部同步到观众端。但是由于观众端屏幕比例可能与主播端不一致。为了完全显示主播端的内容，会进行缩放调整，类似于电影播放时，为了保持原始画面比例并保留原始内容，在某些显示器上，会进行比例缩放，会出现黑边。

## 视角模式 —— 主播，观众，自由（默认）

```Objective-C
typedef NS_ENUM(NSInteger, WhiteViewMode) {
    // 自由模式
    // 用户可以自由放缩、移动视角。
    // 即便房间里有主播，主播也无法影响用户的视角。
    WhiteViewModeFreedom,
    // 追随模式
    // 用户将追随主播的视角。主播在看哪里，用户就会跟着看哪里。
    // 在这种模式中，如果用户进行缩放、移动视角操作，将自动切回 freedom模式。
    WhiteViewModeFollower,
    // 主播模式
    // 房间内其他人的视角模式会被自动修改成 follower，并且强制观看该用户的视角。
    // 如果房间内存在另一个主播，该主播的视角模式也会被强制改成 follower。
    WhiteViewModeBroadcaster,
};

//以下类，只有在 fireRoomStateChanged: 回调事件中，才会使用。
@interface WhiteBroadcastState : WhiteObject
//视角模式
@property (nonatomic, assign) WhiteViewMode viewMode;
@property (nonatomic, assign) NSInteger broadcasterId;
@property (nonatomic, strong) WhiteMemberInformation *broadcasterInformation;
@end

```

### 设置视角模式

* 例子：设置当前用户为主播视角

```
//只需要传入枚举值即可
[whiteRoom setViewMode:WhiteViewModeBroadcaster];
```

### 获取当前视角状态

```Objective-C
[self.room getBroadcastStateWithResult:^(WhiteBroadcastState *state) {
    NSLog(@"%@", [state jsonString]);
}];
```

## 视角中心

同一个房间的不同用户各自的屏幕尺寸可能不一致，这将导致他们的白板都有各自不同的尺寸。实际上，房间的其他用户会将白板的中心对准主播的白板中心（注意主播和其他用户的屏幕尺寸不一定相同）。

我们需要通过如下方法设置白板的尺寸，以便主播能同步它的视角中心。

```Objective-C
[room refreshViewSize];
```

尺寸应该和白板在产品中的实际尺寸相同（一般而言就是浏览器页面或者应用屏幕的尺寸）。如果用户调整了窗口大小导致白板尺寸改变。应该重新调用该方法刷新尺寸。