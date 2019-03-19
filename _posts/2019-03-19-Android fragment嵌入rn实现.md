---
layout: post
title:  "Android fragment嵌入rn实现!"
date:   2019-03-19 13:31:01 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}


First POST build by Jekyll.
github:https://github.com/nishibaiyang


Android fragment嵌入rn实现
------------------------

## 如何实现分析
1. react native调用android native提供给Android端的实现是基于view的接口
2. 在Android中view和fragment是两个不同的概念
3. react native RootComponent最终翻译成native的代码，因此在native侧必须有一个容器（android端Activity、fragment）展示这些elemet

### 应用初始化流程回顾
1. 首先我们会在应用的Application里做RN的初始化操作。

```
//ReactNativeHost：持有ReactInstanceManager实例，做一些初始化操作。
  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    @Override
    public boolean getUseDeveloperSupport() {
      return BuildConfig.DEBUG;
    }

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage()
      );
    }
  };

  @Override
  public ReactNativeHost getReactNativeHost() {
    return mReactNativeHost;
  }

  @Override
  public void onCreate() {
    super.onCreate();
    //SoLoader：加载C++底层库，准备解析JS。
    SoLoader.init(this, /* native exopackage */ false);
  }
}
```

2. 页面继承ReactActivity，ReactActivity作为JS页面的容器。

```
public class MainActivity extends ReactActivity {

    /**
     * Returns the name of the main component registered from JavaScript.
     * This is used to schedule rendering of the component.
     */
    @Override
    protected String getMainComponentName() {
        //返回组件名
        return "standard_project";
    }
}
```

3. 用JS开发页面

```
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

//Component用来做UI渲染，生命周期控制，事件分发与回调。
export default class standard_project extends Component {
  //render函数返回UI的界面结构（JSX编写，编译完成后最终会变成JS代码）
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <Text style={styles.instructions}>
          To get started, edit index.android.js
        </Text>
        <Text style={styles.instructions}> 
          Double tap R on your keyboard to reload,{'\n'}
          Shake or press menu button for dev menu
        </Text>
      </View>
    );
  }
}

//创建CSS样式
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

//注册组件名，JS与Java各自维护了一个注册表
AppRegistry.registerComponent('standard_project', () => standard_project);
```

### 分析ReactActivity
1. ReactActivity继承于Activity，并实现了它的生命周期方法。ReactActivity自己并没有做什么事情，所有的功能都由它的委托类ReactActivityDelegate来完成
2. ReactActivityDelegate
```
public class ReactActivityDelegate {

  protected void onCreate(Bundle savedInstanceState) {
    boolean needsOverlayPermission = false;
    //开发模式判断以及权限检查
    if (getReactNativeHost().getUseDeveloperSupport() && Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
      // Get permission to show redbox in dev builds.
      if (!Settings.canDrawOverlays(getContext())) {
        needsOverlayPermission = true;
        Intent serviceIntent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + getContext().getPackageName()));
        FLog.w(ReactConstants.TAG, REDBOX_PERMISSION_MESSAGE);
        Toast.makeText(getContext(), REDBOX_PERMISSION_MESSAGE, Toast.LENGTH_LONG).show();
        ((Activity) getContext()).startActivityForResult(serviceIntent, REQUEST_OVERLAY_PERMISSION_CODE);
      }
    }

    //mMainComponentName就是上面ReactActivity.getMainComponentName()返回的组件名
    if (mMainComponentName != null && !needsOverlayPermission) {
        //载入app页面
      loadApp(mMainComponentName);
    }
    mDoubleTapReloadRecognizer = new DoubleTapReloadRecognizer();
  }

  protected void loadApp(String appKey) {
    if (mReactRootView != null) {
      throw new IllegalStateException("Cannot loadApp while app is already running.");
    }
    //创建ReactRootView作为根视图,它本质上是一个FrameLayout
    mReactRootView = createRootView();
    //启动RN应用
    mReactRootView.startReactApplication(
      getReactNativeHost().getReactInstanceManager(),
      appKey,
      getLaunchOptions());
    //Activity的setContentView()方法  
    getPlainActivity().setContentView(mReactRootView);
  }
}
```
从loadApp方法中可看出：
1. 创建ReactRootView作为应用的容器，它本质上是一个FrameLayout。
2. 调用ReactRootView.startReactApplication()进一步执行应用启动流程。
3. 通过Java层的UIManagerModule将JS组件转换为Android组件，最终显示在ReactRootView上,
调用Activity.setContentView()将创建的ReactRootView作为ReactActivity的content view。


## 实现方案
1. 将Fragment以静态方式嵌入到布局，加入到ReactRootView（native的）
2. 利用ReactActivity的fragmentmange（即android fragment静态加载和动态加载）

##最终实现，动态加载的方式
1. Native侧需要创建 ViewManager 和 ReactPackage
```
/**
 * @author qiyu
 */
public class FoundManager extends SimpleViewManager<NativeForRNView>{
    @Override
    public String getName() {
        return "FoundPlate";
    }

    @Override
    protected NativeForRNView createViewInstance(final ThemedReactContext reactContext) {
        return new NativeForRNView(reactContext);
    }
}
```

```
/**
 * @author qiyu
 */
public class NativeViewPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(final ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(final ReactApplicationContext reactContext) {
        return Arrays.<ViewManager>asList(
                new FoundManager()
        );
    }
}
```

```
/**
 * @author qiyu
 */
public class NativeForRNView extends FrameLayout {
    private Context mContext;
    private Fragment mCurrentFragment;
    private BaseNewFragment[] mFragments;

    public NativeForRNView(@NonNull final Context context) {
        this(context, null);
    }

    public NativeForRNView(@NonNull final Context context, @Nullable final AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater.from(context).inflate(R.layout.view_found, this);
        mContext = context;
        initView();

    }

    private void initView() {
        switchFragment(FoundFragment.newInstance()).commit();
//        if (mFragments != null && mFragments.length != 0) {
//            switchFragment(FoundFragment.newInstance()).commit();
//            switchFragment(mFragments[0]).commit();
//        }
//        findViewById(R.id.found_plate).requestFocus();
        invalidate();
    }

    private FragmentTransaction switchFragment(Fragment targetFragment) {
        AppCompatActivity activity = getActivity(mContext);
        //activity替换fragment
        if(activity instanceof MainActivity){
            FragmentTransaction transaction = activity.getSupportFragmentManager().beginTransaction();
            if (!targetFragment.isAdded()) {
                if (mCurrentFragment != null) {
                    transaction.hide(mCurrentFragment);
                }
                transaction.add(R.id.fragment_container, targetFragment, targetFragment.getClass().getName());
            } else {
                transaction.hide(mCurrentFragment).show(targetFragment);
            }
            mCurrentFragment = targetFragment;
            return transaction;
        }
        //fragment 替换 fragment，注意宿主fragment必须带R.id.fragment_container
//        if(activity instanceof MainActivity){
//            NewHomeFragment fragment = (NewHomeFragment) ((MainActivity) activity).getHomeFragment();
//            FragmentTransaction transaction = fragment.getChildFragmentManager().beginTransaction();
//            if (!targetFragment.isAdded()) {
//                if (mCurrentFragment != null) {
//                    transaction.hide(mCurrentFragment);
//                }
//                transaction.add(R.id.fragment_container, targetFragment, targetFragment.getClass().getName());
//            } else {
//                transaction.hide(mCurrentFragment).show(targetFragment);
//            }
//            mCurrentFragment = targetFragment;
//            return transaction;
//        }

        FragmentTransaction transaction =activity.getSupportFragmentManager().beginTransaction();
//        FragmentTransaction transaction =((AppCompatActivity)mContext).getSupportFragmentManager().beginTransaction();
//        FragmentTransaction transaction = getChildFragmentManager().beginTransaction();

        if (!targetFragment.isAdded()) {
            if (mCurrentFragment != null) {
                transaction.hide(mCurrentFragment);
            }

//            transaction.add(findViewById(R.id.fragment_container), targetFragment, targetFragment.getClass().getName());
            transaction.add(R.id.fragment_container, targetFragment, targetFragment.getClass().getName());
        } else {
            transaction.hide(mCurrentFragment).show(targetFragment);
        }
        mCurrentFragment = targetFragment;
        return transaction;
    }


    private AppCompatActivity getActivity(Context context) {
        while (!(context instanceof AppCompatActivity) && context instanceof ContextWrapper) {
            context = ((ContextWrapper) context).getBaseContext();
        }

        if (context instanceof Activity) {
            return (AppCompatActivity) context;
        }
        return null;
    }
}
```

2.JS侧AndroidNativeView实现

```
import React, { Component, PropTypes } from 'react';
import {
  requireNativeComponent, View, UIManager,
  findNodeHandle,
} from 'react-native';

const iface = {
  name: 'FoundPlate',
  propTypes: {
    ...View.propTypes,
  },
};

const AndroidNativeView = requireNativeComponent('FoundPlate', iface);
class NativeView extends Component {
  render() {
    return <AndroidNativeView
      {...this.props}
    />;
  }

  create = () => {
    UIManager.dispatchViewManagerCommand(
      findNodeHandle(this.fragment),
      UIManager.TestFragment.Commands.create,
      [], // No args
    );
  }
}

NativeView.propTypes = {

};
module.exports = NativeView;

```

over ~ ~








[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help