#### TextView的setText分析

- setText(CharSequeue text)

- -- setText(CharSequeue text, BufferType type)

- -- setText(CharSequeue text, BufferType type, boolean notifyBefore, int oldlen)

  - text：CharSequence 类型的文本，CharSequence 与 String 都用于定义字符串，但 CharSequence 的是可读可写序列，而 String 的值是只读序列

  - type：字面上是缓存类型，BufferType 是枚举类，枚举值有「NORMAL、SPANNABLE、EDITABLE」

    - SPAN：Android 中提出的概念。无论是 TextView 还是 EditText 显示的其实是文本，Span 是附在文本上的样式，如：前景色、背景色、字体大小等，所有 Span 类都间接继承于 CharSequeue 类
    - SPANNABLE 指在给定的字符区域使用样式
    - EDITABLE 类型类似 StringBuilder，可以追加字符，如，可以在getText 后用 append 追加字符
    - NORMAL 指默认样式

  - notifyBefore：通知之前

  - oldlen：旧文本的长度

  - 源码

    ```java
    //1
    //mTextSetFromXmlOrResourceId指文本是否从xml或resoureceId设置
    mTextSetFromXmlOrResourceId = false;
    if (text == null) {	
        text = "";	
    }
    
    //2
    //If suggestions are not enabled, remove the suggestion spans from the text
    //isSuggestionsEnabled判定InputType是否为特定的几种样式
    if (!isSuggestionsEnabled()) {
        //返回一个CharSequence类型的对象
    text = removeSuggestionSpans(text);	
    }
    
    //3
    //是否设置文本宽度缩放比，若未设置，则设置为1.0f（水平缩放因子默认值，大于1.0f则被拉宽，反之被缩窄）
    if (!mUserSetTextScaleX) mTextPaint.setTextScaleX(1.0f);
    
    //4
    //若text是Spanned的一个实例且文本的开头标记为Spanned的样式	
    //开头为非标记Spanned样式返回 -1 	
    if (text instanceof Spanned && ((Spanned) text).getSpanStart(TextUtils.TruncateAt.MARQUEE) >= 0) {
        //通过ViewConfiguration的get方法来获得上下文对应的isFadingMarqueeEnabled是被google隐藏的函数，是厂商编译framework产生的，用来返回是否用阴影来遮盖文字边缘	
        if (ViewConfiguration.get (mContext).isFadingMarqueeEnabled()){	
            //横向使用阴影来遮盖边缘	
            setHorizontalFadingEdgeEnabled(true);	
            //遮盖模式修改为默认的遮盖	
            mMarqueeFadeMode = MARQUEE_FADE_NORMAL;	
        } else {	
            //不使用横向阴影来遮盖边缘	
            setHorizontalFadingEdgeEnabled(false);	
            //遮盖模式为省略的遮盖	
            mMarqueeFadeMode = MARQUEE_FADE_SWITCH_SHOW_ELLIPSIS;	
        }	
        setEllipsize(TextUtils.TruncateAt.MARQUEE);	
    }
    //内容超出文本区域，通常以省略号的方式显示，在xml中设置android:ellipsize="end"
    
    //5
    //设置文本需要提前经过设置的所有过滤器filter	
    int n = mFilters.length;	
    for (int i=0;i<n;i++) {
        //mFilters为过滤字符集合，遍历mFilters集合，过滤掉text中mFilters中的字符，获取到的新的CharSequence并赋值给text
        CharSequence out = mFilters[i].filter(text, 0, text.length(), EMPTY_SPANNED, 0, 0);	
        if (out != null) {
            text = out;
        }
    }
    
    //6
    //若notifyBefore为true，发送文本改变前的回调
    if (notifyBefore) {	
        if (mText != null) {	
            oldlen = mText.length();	
            sendBeforeTextChanged(mText, 0, oldlen, text.length());	
        } else {	
            sendBeforeTextChanged("", 0, 0, text.length());	
        }	
    }
    //回调
    mTextView.addTextChangedListener(new TextWatcher() {	
        @Override	
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {	
            // 触发回调	
        }
        
        @Override	
        public void onTextChanged(CharSequence s, int start, int before, int count) {}
        
        @Override	
        public void afterTextChanged(Editable s) {}	
    });
    
    //7
    boolean needEditableForNotification = false;	
    if (mListeners != null && mListeners.size() != 0) {	
        needEditableForNotification = true;	
    }	
    //若缓存类型为EDITABLE，getKeyListener不会空且设置addTextChangedListener	
    if (type == BufferType.EDITABLE || getKeyListener() != null || needEditableForNotification) {	
        //创建Editor
        createEditorIfNeeded();	
        mEditor.forgetUndoRedo();	
        //通过mEditableFactory生成Editable对象	
        Editable t = mEditableFactory.newEditable(text);	
        text = t;	
        //设置过滤	
        setFilters(t, mFilters);	
        InputMethodManager imm = InputMethodManager.peekInstance();	
        //打开软键盘	
        if (imm != null) imm.restartInput(this);	
    } else if (type == BufferType.SPANNABLE || mMovement != null) {	
        //mSpannableFactory生成一个新的SpannableString 	
        text = mSpannableFactory.newSpannable(text);	
    } else if (!(text instanceof CharWrapper)) {	
        //根据stringOrSpannedString返回SpannedString或string	
        text = TextUtils.stringOrSpannedString(text);	
    }
    
    //8
    if (mAutoLinkMask != 0) {	
        Spannable s2;
        if (type == BufferType.EDITABLE || text instanceof Spannable) {	
            s2 = (Spannable) text;	
        } else {
            s2 = mSpannableFactory.newSpannable(text);	
        }	
        if (Linkify.addLinks(s2, mAutoLinkMask)) {	
            text = s2;	
            type = (type == BufferType.EDITABLE) ? BufferType.EDITABLE : BufferType.SPANNABLE;	
            //必须在更改movement方法前设置文本，因为setMovementMethod()可能会再次调用setText()来尝试升级缓冲区类型
            mText = text;
            //不要更改支持文本选择的文本的移动方法，可以防止任意光标移动
            if (mLinksClickable && !textCanBeSelected()) {	
                setMovementMethod (LinkMovementMethod.getInstance());	
            }
        }	
    }
    
    //9
    mBufferType = type;	
    mText = text;
    if (mTransformation == null) {	
        mTransformed = text;	
    } else {	
        mTransformed = mTransformation.getTransformation(text, this);	
    }	
    //记录下textLength的长度 	
    final int textLength = text.length();	
    //若text为Spannable实例且不允许转换长度变化	
    if (text instanceof Spannable && !mAllowTransformationLengthChange) {	
        Spannable sp = (Spannable) text;
        //移除所有可能来自其他textview的ChangeWatcher	
        final ChangeWatcher[] watchers = sp.getSpans(0, sp.length(), ChangeWatcher.class);	
        final int count = watchers.length;	
        for (int i = 0; i < count; i++) {	
            // 移除样式	
            sp.removeSpan(watchers[i]);	
        }	
        if (mChangeWatcher == null) mChangeWatcher = new ChangeWatcher();	
        // 设置文本样式	
        sp.setSpan(mChangeWatcher, 0, textLength, Spanned.SPAN_INCLUSIVE_INCLUSIVE | (CHANGE_WATCHER_PRIORITY << Spanned.SPAN_PRIORITY_SHIFT));	
        // 添加SpanWatchers	
        if (mEditor != null) mEditor.addSpanWatchers(sp);	
        if (mTransformation != null) {	
            sp.setSpan(mTransformation, 0, textLength, Spanned.SPAN_INCLUSIVE_INCLUSIVE);	
        }	
        if (mMovement != null) {	
            mMovement.initialize(this, (Spannable) text);	
            //初始化movement方法将设置选择，因此复位mSelectionMoved可以防止它干扰正常的on-focus selection-setting
            if (mEditor != null) mEditor.mSelectionMoved = false;	
        }	
    }
    
    //10
    if (mLayout != null) {
        //内部调用requestLayout()与invalidate()重新布局与绘制
        checkForRelayout();	
    }
    
    //11
    sendOnTextChanged(text, 0, oldlen, textLength);	
    onTextChanged(text, 0, oldlen, textLength);	
    notifyViewAccessibilityStateChangedIfNeeded(AccessibilityEvent.CONTENT_CHANGE_TYPE_TEXT);	
    if (needEditableForNotification) {	
        sendAfterTextChanged((Editable) text);	
    }	
    // SelectionModifierCursorController depends on textCanBeSelected, which depends on text	
    if (mEditor != null) mEditor.prepareCursorControllers();
    //最后发送onTextChanged与afterTextChanged回调
    ```

- 耗时原因：
  - 当textview的宽设置为wrap_content时，底层会调用checkForRelayout函数根据文字多少重新布局
  - 将宽度设置为固定值或者match_parent的时候会大幅度减少绘制时间