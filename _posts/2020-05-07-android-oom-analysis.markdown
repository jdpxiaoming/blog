---
layout: post
title:  "Android内存泄漏追缴真凶-初出茅庐篇"
date:   2020-05-07 19:29:01 +0800
categories: jekyll update
---
Android内存泄漏
===

- 辅助工具[leakCananry](https://github.com/square/leakcanary)
- 辅助工具`AndroidProfile`


### 1. Context上下文泄漏.

- 强引用Activity的Context.

```java
//案例一2020/4/30
public class VersionUtil {
    private Context mContext;
    private static VersionUtil mVersion;

    public VersionUtil(Context context) {
        this.mContext = context;
    }

    public synchronized static VersionUtil getInstance(Context context) {
        if (mVersion == null) {
            mVersion = new VersionUtil(context.getApplicationContext());
        }
        return mVersion;
    }
}

//案例二2020/4/30
public class LoginHelper extends Presenter {
    private Context mContext;
    private static final String TAG = LoginHelper.class.getSimpleName();
    private LogoutView mLogoutView;

    private final String domain = "http://www.ovopark.com:8088/roomservice/weapp/live_room";
    private final MediaType MEDIA_JSON = MediaType.parse("application/json; charset=utf-8");

    private OkHttpClient client;


    public LoginHelper(Context context) {
        mContext = context;

        client = new OkHttpClient.Builder()
                .connectTimeout(5, TimeUnit.SECONDS)
                .readTimeout(5, TimeUnit.SECONDS)
                .writeTimeout(5, TimeUnit.SECONDS)
                .build();

    }

```
- 谁用谁泄露
![fcee78c8.png](/assets/blog_res/fcee78c8.png)



#### 1.1. EventBus使用未回收错误
```
   @Override
    public void onDestroy() {
        super.onDestroy();
        if (mTimeView != null) {
            mTimeView.recycle();
        }

        if (mProgressView != null) {
            mProgressView.onDestroy();
        }
        if (!EventBus.getDefault().isRegistered(this)) {
            EventBus.getDefault().unregister(this);
        }
    }
```

### 1.2. Activity直接回调Fragment的方法（在销毁时候泄漏mContext）
![0ef9a2d1.png](/assets/blog_res/0ef9a2d1.png)

### 1.3 Callback回调阻止回收
![0e8775d7.png](/assets/blog_res/0e8775d7.png)


## 2. Handler 泄漏
1. 在BaseActivity定义的匿名handler
```java
  mHandler = new Handler(getMainLooper()) {
            @Override
            public void handleMessage(Message msg) {
                handlerMessage(msg);
            }
        };
  //子类网络请求后回调了dialog相关操作.
   OkHttpRequest.post(DataManager.Urls.MOBILE_LOGIN, params, new StringHttpRequestCallback() {
            @Override
            public void onSuccess(final String result) {
                try {
                    ResponseData<User> responseData = DataProvider.fasetjsonProviderUserData(LoginActivity.this, result);
                    switch (responseData.getStatusCode()) {
                        case ResponseData.STATUS_CODE_SUCCESS:
                            closeDialog();
                            user = responseData.getResponseEntity().getSuccessEntity();
                            if (null != user) {
                                getSeverAddress(user);
                            }
                            saveUserInfo(mEmailStr, mPasswdStr);
                            break;

                        case ResponseData.STATUS_CODE_USER_UNCHECK:
                            closeDialog();
                            Message messageUncheck = Message.obtain();
                            messageUncheck.what = responseData.getStatusCode();
                            messageUncheck.obj = responseData.getResponseEntity().getFailureMsg();
                            mHandler.sendMessage(messageUncheck);
                            break;

                        case ResponseData.STATUS_CODE_USER_UNPASS:
                            closeDialog();
                            Message messageUnpass = Message.obtain();
                            messageUnpass.what = responseData.getStatusCode();
                            messageUnpass.obj = responseData.getResponseEntity().getFailureMsg();
                            userId = responseData.getResponseEntity().getSuccessEntity().getId();
                            mHandler.sendMessage(messageUnpass);
                            break;

                        default:
                            closeDialog();
                            Message message = Message.obtain();
                            message.what = Constants.Prefs.SHOW_TOAST;
                            message.obj = responseData.getResponseEntity().getFailureMsg();
                            // tab_message.arg1 = responseData.getResponseEntity().getFailureMsg();
                            mHandler.sendMessage(message);
                            break;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    ReportErrorUtils.postCatchedException(e);
                }
            }

            @Override
            public void onFailure(int failureCode, String msg) {
                closeDialog();
                ToastUtil.showToast(LoginActivity.this, msg);
                if (StringUtils.isBlank(msg)) {
                    ToastUtil.showShortToast(LoginActivity.this, getResources().getString(R.string.connect_unknown_failed));
                }
            }
        });
```
- 解决办法使用自定义安全Handler

```java
public class SafeHandler<T extends Context> extends Handler {

    WeakReference<T> mWeakReference ;

    public SafeHandler(T context){
        mWeakReference = new WeakReference<>(context);
    }


    @Override
    public void handleMessage(Message msg) {
        if(mWeakReference.get() == null) return;
    }
}

- invoked 
public void initHandler(){
        mSafeHandler = new SafeHandler<Context>(getContext()){
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);
                if(msg.what == MSG_REPLAY_TIME_OVER){
                    // 尝试结束，需要比对运行时间如果不足30s补足剩余时间 .
                    tryEndCurrentReplay();
                }
            }
        };
  }
```

# 3. MVP中presenter在宿主destroy时候没有销毁http的网络请求.

![1579e494.png](/assets/blog_res/1579e494.png)

- 3.1 销毁网络请求

```java
  /** 取消所有请求请求 */
    public void cancelAll() {
        for (Call call : client.dispatcher().queuedCalls()) {
            call.cancel();
        }
        for (Call call : client.dispatcher().runningCalls()) {
            call.cancel();
        }
    }
  @Override
    public void onDestory() {
        mContext = null;
        mLogoutView = null;
        if(null != client){
            //取消所有网络请求，防止回调.
            cancelAll();
        }
    }
    
    
```

- 3.2 在presenter的每个方法中判断view是否已销毁

```java
if (null != mContext && callBack != null) {
    callBack.loginFailed();
 }
```
# 4. handler.postDelay(xxx_msg,10*1000);

```java
mHandler.sendEmptyMessageDelayed(MSG_GET_USERSIGN, 10 * 1000);

@Override
protected void onDestroy() {
    if (mHandler != null) {
        mHandler.removeCallbacks(permissionRunnable);
        mHandler.removeCallbacks(noticeLayoutRunnable);
        mHandler.removeMessages(MSG_GET_USERSIGN);
    }
}    
```