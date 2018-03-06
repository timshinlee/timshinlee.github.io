Transition框架实际上使用的是Property Animation。

自定义Transition需要继承于Transition类，实现相关方法：
```java
public class CustomTransition extends Transition {
    // 定义属性名，为避免冲突建议使用包名:变换名:属性名的方式命名
    private static final String PROPERTY_NAME_BACKGROUND =
            "com.example.android.customtransition:CustomTransition:background";

    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        // 框架会对开始场景中的每个view调用这个方法
	// TransitionValues中包含了view的引用，以及用来保存view属性值的Map对象
	captureValues(transitionValues);
    }

    @Override
    public void captureEndValues(TransitionValues transitionValues) {
        // 结束场景中的每个view都会调用到这个方法
        captureValues(transitionValues);
    }

    private void captureValues(TransitionValues transitionValues) {
        View view = transitionValues.view;
	transitionValues.values.put(PROPERTY_NAME_BACKGROUND, view.getBackground());
    }

    // 创建自定义的动画，用来应用于场景变换
    @Override
    public Animator createAnimator(ViewGroup sceneRoot, TransitionValues startValues, TransitionValues endValues) {
        // startValues开始场景的值，endValues结束场景的值
        if (startValues == null || endValues == null) {
            return null;
        }
	
	// startValues.view和endValues.view是同个view
        final View target = endValues.view;

        final Drawable startBackground = (Drawable) startValues.values.get(PROPERTY_NAME_BACKGROUND);
        final Drawable endBackground = (Drawable) endValues.values.get(PROPERTY_NAME_BACKGROUND);

        if (startBackground instanceof ColorDrawable && endBackground instanceof ColorDrawable) {
	    // 如果都是颜色背景
            ColorDrawable startColor = (ColorDrawable) startBackground;
            ColorDrawable endColor = (ColorDrawable) endBackground;
            
	    // 如果颜色发生变化
            if (startColor.getColor() != endColor.getColor()) {
	        // 创建动画，Animator在主线程进行，监听过程更新view
                final ValueAnimator animator = ValueAnimator.ofObject(new ArgbEvaluator(), startColor.getColor(), endColor.getColor());
                animator.setDuration(2000);
                animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                    @Override
                    public void onAnimationUpdate(ValueAnimator animation) {
                        final Object animatedValue = animation.getAnimatedValue();
                        if (null != animatedValue) {
                            target.setBackgroundColor((Integer) animatedValue);
                        }
                    }
                });
                return animator;
            }
        }
	// 返回空表示不进行动画
        return null;
    }
}
```

定义好的Transition子类就可以直接实例化使用了，据测试在使用Scene的Transition方式中可以正常使用，在delayedTransition貌似不能使用。
