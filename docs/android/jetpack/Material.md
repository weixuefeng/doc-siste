---
Author: pony@diynova.com
Date: 2022-02-23 14:30:11
LastEditors: pony@diynova.com
LastEditTime: 2022-02-24 11:53:49
FilePath: /notes/docs/android/jetpack/Material.md
Description: 
---

# Material System Documentation

## [Bottom navigation](https://material.io/components/bottom-navigation)

- navigation bug, 需要研究具体原因

如下配置无法正确导航到 homeFragment
```
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/bottom_nav_graph"
    app:startDestination="@+id/navigationHome"
    >
    <fragment
        android:id="@+id/navigationHome"
        android:name="org.wave.ui.HomeFragment"
        android:label="@string/home"
        tools:layout="@layout/fragment_home"
        />
    <fragment
        android:id="@+id/navigation_dashboard"
        android:name="org.wave.ui.CameraFragment"
        android:label="@string/camera"
        tools:layout="@layout/fragment_camera"
        />
    <fragment
        android:id="@+id/navigation_notifications"
        android:name="org.wave.ui.MeFragment"
        android:label="@string/me"
        tools:layout="@layout/fragment_me"
        />
</navigation>
```

正确配置
```
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/bottom_nav_graph"
    app:startDestination="@+id/navigation_home"
    >
    <fragment
        android:id="@+id/navigation_home"
        android:name="org.wave.ui.HomeFragment"
        android:label="@string/home"
        tools:layout="@layout/fragment_home"
        />
    <fragment
        android:id="@+id/navigation_dashboard"
        android:name="org.wave.ui.CameraFragment"
        android:label="@string/camera"
        tools:layout="@layout/fragment_camera"
        />
    <fragment
        android:id="@+id/navigation_notifications"
        android:name="org.wave.ui.MeFragment"
        android:label="@string/me"
        tools:layout="@layout/fragment_me"
        />
</navigation>
```
