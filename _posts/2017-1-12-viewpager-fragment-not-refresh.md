---
layout:     post
title:      "ViewPager中动态更换Fragment,调用notifyDataSetChanged()方法Fragment不更新的问题及解决方案"
date: 2017-03-16 10:41:10 +0800
categories: Android
tags: Android adapter ViewPager
author: yang1006
---
* content
{:toc}
ViewPager中动态更换Fragment,调用notifyDataSetChanged()方法Fragment不更新的问题及解决方案。




  <p><font color="#464646">最近在使用 ViewPage+FragmentPagerAdapter 时遇到一个问题，就是在 Fragment 数据集合改变时候，调用adapter的 notifyDataSetChanged() 方法 Fragement 根本不刷新。</font></p>

### 原因

<p>研究<font color="000000"> FragmentPagerAdapter.instantiateItem </font> 源码发现：</p>

	@Override
    public Object instantiateItem(ViewGroup container, int position) {
        if (mCurTransaction == null) {
            mCurTransaction = mFragmentManager.beginTransaction();
        }

        final long itemId = getItemId(position);

        // Do we already have this fragment?
        String name = makeFragmentName(container.getId(), itemId);
        Fragment fragment = mFragmentManager.findFragmentByTag(name);
        if (fragment != null) {
            if (DEBUG) Log.v(TAG, "Attaching item #" + itemId + ": f=" + fragment);
            mCurTransaction.attach(fragment);
        } else {
            fragment = getItem(position);
            if (DEBUG) Log.v(TAG, "Adding item #" + itemId + ": f=" + fragment);
            mCurTransaction.add(container.getId(), fragment,
                    makeFragmentName(container.getId(), itemId));
        }
        if (fragment != mCurrentPrimaryItem) {
            fragment.setMenuVisibility(false);
            fragment.setUserVisibleHint(false);
        }

        return fragment;
    }

<p><font color="#464646">在 instantiateItem 的时候，生成的 fragment 会存到 <font color="000000">FragmentManager</font> 中，下次再 instantiateItem 同一位置的 item 时候，会根据名字在 FragmentManager 寻找是否有缓存的 fragment，如果有的话就会直接只用缓存的 fragment，这就是导致 Fragment 数据集改变之后调用 notifyDataSetChanged() 方法也不会刷新的原因。</font></p>
  
### 解决方案
<p><font color="#464646">1、在设置了新的 Fragment 数据集之后，设置标记位，在 instantiateItem 中替换 FragmentManager 缓存的 fragment，代码如下：</font></p>

	
	private ArrayList<Fragment> fragments;
    private FragmentManager fm;
    private boolean[] flags;//标识,重新设置fragment时全设为true
    
	@Override
    public Object instantiateItem(ViewGroup container, int position) {
        if (flags != null && flags[position]) {
            /**得到缓存的fragment, 拿到tag并替换成新的fragment*/
            Fragment fragment = (Fragment) super.instantiateItem(container, position);
            String fragmentTag = fragment.getTag();
            FragmentTransaction ft = fm.beginTransaction();
            ft.remove(fragment);
            fragment = fragments.get(position);
            ft.add(container.getId(), fragment, fragmentTag);
            ft.attach(fragment);
            ft.commit();
            /**替换完成后设为false*/
            flags[position] = false;
            if (fragment != null){
                return fragment;
            }else {
                return super.instantiateItem(container, position);
            }
        } else {
            return super.instantiateItem(container, position);
        }
    }
    
    @Override
    public int getItemPosition(Object object) {
        return POSITION_NONE;
    }
    
    public void setFragments(ArrayList<Fragment> fragments) {
        if (this.fragments != null) {
            flags = new boolean[fragments.size()];
            for (int i = 0; i < fragments.size(); i++) {
                flags[i] = true;
            }
        }
        this.fragments = fragments;
        notifyDataSetChanged();
    }

<p><font color="#464646">这其中还有一个问题要注意的就是需要重写<font color="000000"> getItemPosition() 方法并返回 POSITION_NONE</font>。
参见源码可以发现 notifyDataSetChanged() 方法会回调 ViewPager 的dataSetChanged()方法，在该方法中会根据 getItemPosition() 的返回值进行判断，如果是 POSITION_UNCHANGED (默认返回)则什么都不做直接 continue，如果是 POSITION_NONE，则会调用 PagerAdapter.destroyItem() 并设置 needPopulate 为 true，才会触发 instantiateItem() 方法去生成新的对象。</font></p>

<p><font color="#464646">2、因为 FragmentPagerAdapter.instantiateItem() 中 mFragmentManager 是通过 name 去寻找是否有缓存的 fragment:</font></p>

		final long itemId = getItemId(position);
		String name = makeFragmentName(container.getId(), itemId);
		Fragment fragment = mFragmentManager.findFragmentByTag(name);
		
		private static String makeFragmentName(int viewId, long id) {
        	return "android:switcher:" + viewId + ":" + id;
    }

<p><font color="#464646">而 name 是根据 viewId 和 getItemId(position) 确定的，我们可以通过修改<font color="000000"> getItemId(position) </font>方法来让   mFragmentManager 找不到 fragment 从而去使用新的 fragment 替换。getItemId(position) 默认返回的是 position，我这里使用 position+时间戳来替换。</font></p>

	private long time;
	public void setFragments(ArrayList<Fragment> fragments) {
        time = System.currentTimeMillis();
        this.fragments = fragments;
        notifyDataSetChanged();
    }
    
    @Override
    public long getItemId(int position) {
        return super.getItemId(position) + time;
    }

<p><font color="#464646">两种方法都亲测有效，当然大家也可以继续研究源码使用其他方法。</font></p>




参考文档：
[http://www.cnblogs.com/dancefire/archive/2013/01/02/why-notifydatasetchanged-does-not-work.html](http://www.cnblogs.com/dancefire/archive/2013/01/02/why-notifydatasetchanged-does-not-work.html)



	




	
