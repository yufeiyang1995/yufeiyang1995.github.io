---
layout:     post
title:      在Android应用中错误提示的几种实现方法
category: blog
description: 在开发安卓应用时会经常有需要提示输入错误这样的场景，因此我在实验中对几种提示信息的方法都进行了一些尝试
---

## 在Android应用中错误提示的几种实现方法

举个例子，我在实现时想要这样一个效果：用户注册时输入密码，我希望界面可以在他输入不合规定的密码时就在输入框后面显示一个错误提示，并在他更换输入框后也会留有一个错误标记在原来的输入框后面。带着这样一个想法开始了我对代码的实现过程，没想到探索出了好几种实现方法，最终使用了最适合我的一种。

#### 1. OnClick()方法

这种方法是最简单点，也最常用的方法了，这里的实现也最简单，在点击事件里才对密码的格式进行检查，然后不符合规定进行一定方式的提示(DialogAlert or Toast)，让用户重新输入。这样的设计并不聪明，也不是很符合我们的要求，这里只是简单提一下这一常用方法，就不做具体介绍了，也不推荐使用，因为强大的安卓提供了很多更好的方法供我们使用~~~

#### 2. OnFocusChange()方法

这种方法利用的是获取焦点的方法来触发响应事件。比如，当我们输入完密码切换到其他输入框时，这时就可以对密码的输入情况进行一个判断，从而完成在一个输入框输入结束之后进行提示的效果。但是这里在实现中发现了一个问题，就是在输入框加一个SetError()函数作为错误显示的时候，没有像预想的那样在该密码输入框后面出现一个红色感叹号，所以我使用了Toast来进行提示，但是组长说这样的提示不合要求，从而导致我放弃了这个做法。

但是，后来优化界面之后，修改了一下控件的使用发现可能是引入了support控件的原因导致感叹号没有显示，也就是说这种方法是可行的。

OnFocusChange()方法使用方法如下：

```
控件名.setOnFocusChangeListener(new View.OnFocusChangeListener()
{
    //v 发生变化的视图    hasFocus:用来判断视图是否获得了焦点
    public void onFocusChange(View v,boolean hasFocus)
    {
        EditText _v = (EditText)v;
        if(!hasFocus){}
        else{}
    }
});
```

#### 3. addTextChangedListener()方法

最后，和组长确认之后，组长觉得实时监听输入的正确性使我们要的效果，因此最后采用了这个实时监听输入的方法。修改了控件之后发现setError()有了正常的效果，一下子解决了两个问题，提前收工，2333~

下面用例子进行说明该方法：

```
mEmailView.addTextChangedListener(new TextWatcher() {
    private CharSequence temp;
    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        temp = s;
    }
    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
    }
    @Override
    public void afterTextChanged(Editable s) {
	    if (!mEmailView.getText().toString().equals("") && !isEmailValid(mEmailView.getText().toString())){
            mEmailView.setError(getString(R.string.error_invalid_email));
            mEmailView.requestFocus();
        }
	}
});
```

这个方法是每一次输入一个字符时都会进行调用的方法，从它的内部函数名中就可以看出来，这三个函数分别表示输入前，输入中和输入后，在这里，我们只需要对每次输入后进行对其格式的判断，如果不符合则在这个控件下加一个SetError提示，并且加入requestFocus()要求，表示当交点改变时依然会有一个感叹号提示（其实第二种方法也是这样实现的，但是当时的控件有一些问题，所以没有出现更换焦点仍有提示的效果，所以现在我觉得第二种方法实现也是一个不错的效果）。这样就完成了我们要求的实现。

#### 总结

做完这一系列的尝试，也算是了解了一套EditText的监听机制，我个人觉得第二种实现是最好的一种实现，不会实时监听浪费资源，还可以达到一个比较好的提示效果。