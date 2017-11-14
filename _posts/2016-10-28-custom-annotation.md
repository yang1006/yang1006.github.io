---
layout:     post
title:      ANDROID 自定义注解
category: blog
description: 通过自定义注解实现findViewById方法
---

  在开发中我们经常会使用到findViewById方法初始化一个控件，如果一个布局里控件太多就会非常浪费时间，不能专注于核心逻辑的实现。本篇文章主要就是讲解通过注解实现findViewById方法的功能。
  
  android中常见的注解有Override和SuppressWarnings等，其中Override表明这个方法是复写了父类的方法，SuppressWarnings表示屏蔽一些警告。
  
  接下来我们来自定义一个注解实现替换findViewById方法。自定义注解分为2步，定义和实现。
  首先定义一个注解：
  		
	@Target(ElementType.FIELD)
	@Retention(RetentionPolicy.RUNTIME)
	public @interface InjectView {
	  /**控件的id*/
	  int id() default -1;
	}

  其中Target表示这个注解是用在哪个地方的，它的值是一个枚举类型：
  		
1. TYPE               描述类、接口和枚举类型
2. FIELD              描述字段
3. METHOD             描述方法
4. PARAMETER          描述参数
5. CONSTRUCTOR        描述构造方法
6. LOCAL_VARIABLE     描述局部变量
7. ANNOTATION_TYPE    描述注解
8. PACKAGE            描述包
  		
  Retention用于描述你的注解的生命周期：
  
1. SOURCE  注解只在源码中有效
2. CLASS   注解在源码和class文件中有效(默认)
3. RUNTIME 注解在源码、class文件和运行时都有效
  
  所以我们自定义的注解用于描述字段，并且是在运行时有效。
  接下来我们定义一个基类Activity，在基类的onCreate方法中解析InjectView:
  
	public abstract class BaseActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getLayoutId());
        analyseInjectView();
    }

    /**返回布局id*/
    protected abstract int getLayoutId();

    /**根据注解自动解析控件*/
    private void analyseInjectView(){
        try {
            Class clazz = this.getClass();
            Field[] fields = clazz.getDeclaredFields();
            for (Field field : fields){
                InjectView injectView = field.getAnnotation(InjectView.class);
                if (injectView != null){
                    int id = injectView.id();
                    Log.e("yang1006", "id->"+id);
                    if (id > 0){
                        field.setAccessible(true);
                        field.set(this, findViewById(id));
                    }
                }
            }
        }catch (Exception e){}
    }
	}
	
 最后就是继承BaseActivity 使用InjectView了:
 		
	public class CustomAnnotationActivity extends BaseActivity {
    	@Override
    	protected int getLayoutId() {
     	   return R.layout.activity_custom_annotation;
    	}

    	@InjectView(id = R.id.tv_1)
    	  private TextView tv_1;


    	@Override
    	protected void onResume() {
       	  super.onResume();
		  tv_1.setText("已成功代替findViewById方法");
          }
	}
	
  这样几行代码就可以了，相比之前使用findViewById的方式可以提升开发的效率。