---
description: LocalAuthentication
---

# 本地身份验证（TouchID & FaceID）

Tag: `生物认证`｜`指纹`\|`面容`\|`Touch ID`｜`Face ID`｜`LAContext`

Platform: iOS 8 ~ iOS 13

### LAContext

首先，我们需要创建并配置一个认证上下文`LAContext *context`来验证用户身份。现有的验证方法包含 Touch ID 、 Face ID 和 锁屏密码。苹果将这3种验证方法划分为2种验证方式（LAPolicy）。我们可以根据业务场景选择适合的验证方式。

其次，在身份认证前，需要先调用`canEvaluatePolicy:error:`判断身份验证方式（LAPolicy）是否可行

* **验证方式可行**：返回 YES
* **验证方式不可行**：返回 NO，参数 `error` 保存错误信息（LAError）

该方法调用后可判断机器支持的生物认证类型`(LABiometryType)context.biometryType`（iOS11以上），iOS11以下且返回YES的 都支持指纹 Touch ID

* **LABiometryTypeTouchID**：本机支持指纹 Touch ID
* **LABiometryTypeFaceID**：本机支持面容 Face ID
* **LABiometryNone**：本机不支持生物认证

```text
- (BOOL)canEvaluatePolicy:(LAPolicy)policy error:(NSError * __autoreleasing *)error
```

接着，在`canEvaluatePolicy:error:`方法返回YES后，我们还可以判断 Touch ID 或 Face ID 是否已经变更（iOS9.0以上）。判断 Touch ID 或 Face ID 是否变化需要使用到 `(NSData *)context.evaluatedPolicyDomainState`。使用方法简单来说，就是在上次生物验证成功后，将该参数缓存在本地，与本次新获得的参数进行比较，一致则没有变化，不一致则说明有变化。

> 注意：`evaluatedPolicyDomainState`在`canEvaluatePolicy:error:`返回YES后或`evaluatePolicy:localizedReason:reply:`调用后，才能在context中获取。

```text
- (void)evaluatePolicy:(LAPolicy)policy
       localizedReason:(NSString *)localizedReason
                 reply:(void(^)(BOOL success, NSError * __nullable error))reply
```

最后，我们调用`evaluatePolicy:localizedReason:reply:`来弹出身份验证窗口，等待用户操作后执行异步回调`replay`

* **身份验证成功**： 回调 `replay` ，block 参数 `success` 为 YES
* **身份验证失败**： 回调 `replay` ，block 参数 `success` 为 NO，参数 `error` 保存错误信息（LAError）

#### LAPolicy

现有的验证方法包含 Touch ID 、 Face ID 和 锁屏密码。苹果将这3种验证方法划分为以下2种验证方式。

```text
LAPolicyDeviceOwnerAuthenticationWithBiometrics // API_AVAILABLE(ios(8.0))
LAPolicyDeviceOwnerAuthentication // API_AVAILABLE(ios(9.0))
```

* **LAPolicyDeviceOwnerAuthenticationWithBiometrics**

身份认证（仅使用生物认证，例如 Touch ID 和 Face ID）

* **LAPolicyDeviceOwnerAuthentication**

身份认证（可使用锁屏密码或者生物认证， 例如 Touch ID 、 Face ID 和 锁屏密码）

> **注意**：该方法可绕过生物认证，仅使用锁屏密码即可通过认证。
>
> **小技巧（iOS9 以上）**：使用 `LAPolicyDeviceOwnerAuthenticationWithBiometrics`验证方式在用户多次尝试生物认证失败锁定后，可调用`LAPolicyDeviceOwnerAuthentication`验证方式让用户输入锁屏密码解锁，而后再次调用 `LAPolicyDeviceOwnerAuthenticationWithBiometrics`验证方式进行生物认证。

#### LAError

以下是所有的错误码和相应的错误原因。

| LAError | Error Macro | Error codes |
| :--- | :--- | :--- |
| LAErrorAuthenticationFailed | kLAErrorAuthenticationFailed | -1 |
| LAErrorUserCancel | kLAErrorUserCancel | -2 |
| LAErrorUserFallback | kLAErrorUserFallback | -3 |
| LAErrorSystemCancel | kLAErrorSystemCancel | -4 |
| LAErrorPasscodeNotSet | kLAErrorPasscodeNotSet | -5 |
| LAErrorTouchIDNotAvailable（iOS8 - iOS10） | kLAErrorTouchIDNotAvailable | -6 |
| LAErrorTouchIDNotEnrolled（iOS8 - iOS10） | kLAErrorTouchIDNotEnrolled | -7 |
| LAErrorTouchIDLockout（iOS8 - iOS10） | kLAErrorTouchIDLockout | -8 |
| LAErrorAppCancel（iOS9- iOS13） | kLAErrorAppCancel | -9 |
| LAErrorInvalidContext（iOS9- iOS13） | kLAErrorInvalidContext | -10 |
| LAErrorBiometryNotAvailable（iOS11 - iOS13） | kLAErrorTouchIDNotAvailable | -6 |
| LAErrorBiometryNotEnrolled（iOS11 - iOS13） | kLAErrorTouchIDNotEnrolled | -7 |
| LAErrorBiometryLockout（iOS11 - iOS13） | kLAErrorTouchIDLockout | -8 |
| LAErrorNotInteractive | kLAErrorNotInteractive | -1004 |

```text
LAErrorAuthenticationFailed // 身份验证失败

LAErrorUserCancel // 身份验证被用户手动取消

LAErrorUserFallback // 用户不进行身份认证，选择替代方法（相关配置：LAContext的localizedFallbackTitle，默认为“输入密码”，传入空字符串则隐藏该按钮）

LAErrorSystemCancel // 身份认证被系统取消 (如遇到来电)

LAErrorPasscodeNotSet // 用户没有设置锁屏密码

LAErrorTouchIDNotAvailable // TouchID无法使用（系统不支持TouchID）

LAErrorTouchIDNotEnrolled // TouchID未配置（用户没有配置指纹）

LAErrorTouchIDLockout // TouchID被锁定（用户连续失败次数过多被锁定）

LAErrorAppCancel // 身份认证过程中LAContext调用了Invalidated

LAErrorInvalidContext // 身份认证前LAContext调用了Invalidated

LAErrorBiometryNotAvailable // 生物认证无法使用（系统不支持指纹或面容）

LAErrorBiometryNotEnrolled //  生物认证未配置（用户没有配置指纹或面容）

LAErrorBiometryLockout // 生物认证被锁定（用户连续失败次数过多被锁定）

LAErrorNotInteractive // 身份认证弹窗失败（使用LAContext的interactionNotAllowed禁用交互）
```

