title: Android6.0 Telephone 源码分析（一）
date: 2016-07-22 23:32:04
tags:
  - Android
---
### DialtactsActivity
1、点击telephone APP ，通过抓log可以清楚的看到程序入口是DialtactsActivity.onClick()方法:
![2](https://raw.githubusercontent.com/XianboChen/MyBlog/master/picture/2.png)
``` java
public void onClick(View view) {

        switch (view.getId()) {//对触发的buttonID进行判断
            case R.id.floating_action_button://点击的是“拨号”按钮
                 if(mListsFragment.getCurrentTabIndex()==ListsFragment.TAB_INDEX_ALL_CONTACTS&&!mInRegularSearch) {       //通过电话簿拨号
                  DialerUtils.startActivityWithErrorToast(this,IntentUtil.getNewContactIntent(), R.string.add_contact_not_available);
                }else if (!mIsDialpadShown) {//拨号界面未显示，显示拨号界面
                    mInCallDialpadUp = false;
                    showDialpadFragment(true);//在该方法中newl了DialpadFragment，实际上也就是交给了DialPadFragment去处理
                }
        break;

            case R.id.voice_search_button://点击的是“声控搜索”按钮

                try {

                    ...  //略
                }

                break;

            case R.id.dialtacts_options_menu_button://设置按钮

              ....//略

                break;

            default: {

              ....//略

                break;
            }
       }
    }

```
### DialPadFragment
2、调用showDialpadFragment()方法，在该方法中new出了DialPadFragment,此时进入类DialPadFragment中的onClick()方法（这里有两个DialPadFragment.java文件，一个在./dialpad文件夹中，还有一个在./incall文件夹中，这是在来电时调用的。
```   java
 @Override

    public boolean onKey(View view, int keyCode, KeyEvent event) {
        switch (view.getId()) {
           case R.id.digits:
                if (keyCode == KeyEvent.KEYCODE_ENTER) {//如果是KEYCODE_ENTER，开始拨打
                    handleDialButtonPressed();
                    return true;//自己处理了
                }
            break;
        }
        return false;//返回false由系统进行处理
    }

```


如果点击的是enter建，将会用handleDialButtonPressed()方法:
>  /*
>In most cases, when the dial button is pressed, there is a
>number in digits area. Pack it in the intent, start the
>outgoing call broadcast as a separate task and finish this activity.
>  When there is no digit and the phone is CDMA and off hook,
 > resending a blank flash for CDMA. CDMA networks use Flash
   > messages when special processing needs to be done, mainly for
  >  3-way or call waiting scenarios. Presumably, here we&#39;re in a
> special 3-way scenario where the network needs a blank flash
 > before being able to add the new participant.  (This is not the
> case with all 3-way calls, just certain CDMA infrastructures.)
>
> Otherwise, there is no digit, display the last dialed number. Don&#39;t finish since the user may want to edit it. The
>user needs to press the dial button again, to dial it (general
>  case described above).

> */







*在大多数情况下，当拨号按钮被按下时都会有一串数字存在。从它的intent对象中得到它，开始

*呼叫广播作为一个单独的任务，并结束这个任务活动。

*如果没有数字并且CDMA网络处于占线状态时，

*我们发送一个空白的Flash CDMA。CDMA网络使用Flash

*来处理特殊处理的消息时，这里主要有三种方式

*以及呼叫等待的情景。这里我们应该属于

*前者，在能够添加新参与者之前,网络需要一个空白的Flash。

*若不是上面的情景，也不要结束这个Activity，应该显示上一次的电话号码

*并等待用户可能要编辑它。

*用户需要再次按拨号键，重新拨通（重复上面描述的案例）。

 */


-----------  部分翻译可能不够准确  ------------

``` java

private void handleDialButtonPressed() {

        if (isDigitsEmpty()) { //如果没有数字输入

            handleDialButtonClickWithEmptyDigits();

        } else {

           final String number = mDigits.getText().toString();                        //获取输入的电话号码

            if (number != null
                    && !TextUtils.isEmpty(mProhibitedPhoneNumberRegexp)
                    && number.matches(mProhibitedPhoneNumberRegexp)) {                         //对number过滤
                Log.i(TAG, "The phone number is prohibited explicitly by a rule.");
               if (getActivity() != null) {
                    DialogFragment dialogFragment = ErrorDialogFragment.newInstance(
                            R.string.dialog_phone_call_prohibited_message);
                    dialogFragment.show(getFragmentManager(), "phone_prohibited_dialog");

                }

                clearDialpad();                                            //清除数字

            } else {                                              //如果number通过了检查

                final Intent intent = CallUtil.getCallIntent(number,                  //取得一个Callintent


                        (getActivity() instanceof DialtactsActivity ?
                               ((DialtactsActivity) getActivity()).getCallOrigin() : null));

                              DialerUtils.startActivityWithErrorToast(getActivity(), intent);   //交给DialerUtils去处理
                hideAndClearDialpad(false)
            }
        }
    }

```

* ~~如果用户点击的不是enter键，onKey方法将会返回false,交给onPressed处理~~（*这里暂存疑 *）。

```  java

public void onPressed(View view, boolean pressed) {

        if (DEBUG) Log.d(TAG, "onPressed(). view: " + view + ", pressed: " + pressed);

        if (pressed) {                        //这里继续对各个按键判断
            switch (view.getId()) {
                case R.id.one: {
                    keyPressed(KeyEvent.KEYCODE_1);  //在keyPressed()方法中调用playTone（）方法，从而产生不同的按键声音效果
                    break;
                }

               ... 略

                case R.id.zero: {
                    keyPressed(KeyEvent.KEYCODE_0);
                    break;
                }

                case R.id.pound: {
                    keyPressed(KeyEvent.KEYCODE_POUND);
                    break;
                }

                case R.id.star: {
                    keyPressed(KeyEvent.KEYCODE_STAR);
                    break;
                }

                default: {
                    Log.wtf(TAG, "Unexpected onTouch(ACTION_DOWN) event from: " + view);
                    break;
                }
            }
            mPressedDialpadKeys.add(view);
        } else {
            mPressedDialpadKeys.remove(view);
           if (mPressedDialpadKeys.isEmpty()) {
                stopTone();
            }
        }
    }

```

随后继续触发onClick方法：

``` java

    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.dialpad_floating_action_button:                                  //拨打按钮
                mHaptic.vibrate();
              handleDialButtonPressed();                                    //再次交给了handleDialButton()方法
                break;

            case R.id.deleteButton: {                                                  //del按钮
                keyPressed(KeyEvent.KEYCODE_DEL);
                break;
            }

           case R.id.digits: {                                             //数字显示框，聚焦

                if (!isDigitsEmpty()) {
                   mDigits.setCursorVisible(true);
                }
                break;

            }

            case R.id.dialpad_overflow: {
                mOverflowPopupMenu.show();
                break;
            }

            default: {
               Log.wtf(TAG, "Unexpected onClick() event from: " + view);
               return;
            }
        }
    }

```
**总之，在类DialPadFragment中完成了拨号，并将带着拨号数字的intent交给了类DialerUtils。 **
> DialerUtils.startActivityWithErrorToast(getActivity(), intent);


### DialerUtils
3、进入类DialerUtils.startActivityWithErrorToast()方法，这里有多个重载：

```  java

public static void startActivityWithErrorToast(Context context, Intent intent, int msgId) {
        try {                                         //判断Intent是否是ACTION_CALL
            if (Intent.ACTION_CALL.equals(intent.getAction())) {
                // All dialer-initiated calls should pass the touch point to the InCallUI
                Point touchPoint = TouchPointManager.getInstance().getPoint();
               if (touchPoint.x != 0 || touchPoint.y != 0) {
                    Bundle extras = new Bundle();
                    extras.putParcelable(TouchPointManager.TOUCH_POINT, touchPoint);
                    intent.putExtra(TelecomManager.EXTRA_OUTGOING_CALL_EXTRAS, extras);
               }

                  final TelecomManager tm = (TelecomManager)            //获取TelecomManager服务
                   context.getSystemService(Context.TELECOM_SERVICE);
                  tm.placeCall(intent.getData(), intent.getExtras());
            } else {
                context.startActivity(intent);
            }
        } catch (ActivityNotFoundException e) {
            Toast.makeText(context, msgId, Toast.LENGTH_SHORT).show();
        }
    }
```
### TelecomManager
4、进入类TelecomManager.placeCall()方法,此时已经在framework包下，进入了framework层：
``` java
public void placeCall(Uri address, Bundle extras) {
        ITelecomService service = getTelecomService();//获取TelecomService服务
        if (service != null) {
            if (address == null) {
                Log.w(TAG, "Cannot place call to empty address.");
            }
            try {
                service.placeCall(address, extras == null ? new Bundle() : extras,
                        mContext.getOpPackageName());//交给Service去处理，进入Server端
            } catch (RemoteException e) {
                Log.e(TAG, "Error calling ITelecomService#placeCall", e);
            }
        }
    }
```
----


































