### DataBinding

- 编译阶段，DataBinding介入，扫描res/layout/下所有布局文件，生成相应的ViewBinding抽象类、实现类
  
  - 生成两个XML文件，一个和原布局类似，多了tag标签，一个根据tag标签获取对应控件值
  
- 步骤

  - `ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main)`
  - DataBindingUtil
    - setContentView():T
    - bindToAddViews():T
    - bind():T
  - DataBinderMapperImpl
    - getDataBinder:ViewDataBinding 获取tag标签
  - ActivityMainBindingImpl
    - 传入ViewDataBinding.mapBindings:Object[]，数组大小和布局控件对象数量一致，解析xml的tag获取布局，不用再findviewbyid
  - BR 记录注册@Bindable的属性
  - `binding.setUser(user)`
  - ActivityMainBindingImpl
    - setUser()，mDirtyFlag |= 0x1L
  - ViewDataBinding 继承 BaseObservable
    - updateRegistration():boolean，传入ViewDataBinding.CREATE_PROPERTY_LISTENER:CreateWeakListener，从mLocalFieldIdObservers数组中拿缓存的监听器WeakListener，数组下标是BR属性
    - 如果拿不到缓存，registerTo()
    - WeakListener.setTarget(observable)，observable是User类
    - mObservable.addListener(mTarget)，添加到ArrayList中
    - notifyPropertyChanged()
  - BaseObservable
    - notifyPropertyChanged()
  - CallbackRegistry
    - notifyCallbacks()
  - PropertyChangeRegistry
  - ViewDataBinding
    - onPropertyChanged()
    - binder.handleFieldChange()，处理字段改变
  - ActivityMainBindingImpl
    - onFieldChange()
    - onChangeUser():boolean，位操作标识mDirtyFlags，二进制作为标记，标记哪个字段要修改，超过了用两个变量组合
  - ViewDataBinding
    - requestRebind()，做下版本适配
    - mUIThreadHandler.post(mRebindRunnable)，转到主线程
    - executeBindingsInternal()
  - ActivityMainBindingImpl
    - executeBindings()，拿到User对象，根据dirtyFlags修改
  - TextViewBindingAdapter
    - setText()，view.setText()

  ![339705c3da1276f2ad93c8ad68f3503](C:\Users\13085\AppData\Local\Temp\WeChat Files\339705c3da1276f2ad93c8ad68f3503.png)

  

![image-20210719093650804](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210719093650804.png)

![image-20210719155214431](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210719155214431.png)