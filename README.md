---
description: >-
  说明符（Specifier）是用来修饰 UClass, UFunction, UProperty, UEnum, UInterface, UStruct
  的关键词，定义代码与编辑器之间的交互方式。
---

# 🕹 说明符 (Specifiers)

官方文档：

{% embed url="https://docs.unrealengine.com/5.2/en-US/class-specifiers" fullWidth="false" %}

{% embed url="https://docs.unrealengine.com/5.2/en-US/unreal-engine-uproperty-specifiers/" %}







## 一、常用 Class Specifiers

### 1.1 Class Specifier

<table><thead><tr><th width="216">Class Specifier</th><th>作用</th></tr></thead><tbody><tr><td>Abstract</td><td>声明该类是抽象类，不能在关卡里实例化，<mark style="color:orange;"><strong>不会被继承</strong></mark></td></tr><tr><td>Blueprintable</td><td>声明该类可以作为蓝图的基类，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>NotBlueprintable</td><td>声明该类不能作为蓝图的基类，这是默认的设置并且<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>BlueprintType</td><td>声明该类可以作为蓝图创建的变量类型，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>Const</td><td>声明该类所有属性和函数都是 const，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>Deprecated</td><td>声明该类已被弃用，其对象在序列化时不会保存，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>MinimalAPI</td><td>声明该类只能被 Cast To，但是不能使用其中的任何函数（除了 Inline 函数）。<br>能提高编译速度，因为不需要给其他模块导出所有功能</td></tr><tr><td>Placeable</td><td>声明该类可以被放置到关卡、UI、蓝图里，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>NotPlaceable</td><td>取消 Placeable 的设置，<mark style="color:orange;"><strong>不会被继承</strong></mark></td></tr><tr><td>Transient</td><td>声明该类是临时的，其对象不会保存到磁盘，<mark style="color:purple;"><strong>会被继承</strong></mark></td></tr><tr><td>NonTransient</td><td>取消 Transient 的设置，<mark style="color:orange;"><strong>不会被继承</strong></mark></td></tr></tbody></table>

### 1.2 Class Metadata Specifier

Class Metadata Specifier 是只存在于 Editor 的数据，用来对 UClass 补充声明，用法示例：

{% code overflow="wrap" %}
```cpp
UCLASS(BlueprintType, Blueprintable, config=Engine, meta=(ShortTooltip="An Actor is an object that can be placed or spawned in the world."))
class ENGINE_API AActor : public UObject
{
}
```
{% endcode %}

<table><thead><tr><th width="301">Class Meta Tag</th><th>作用</th></tr></thead><tbody><tr><td>BlueprintSpawnableComponent</td><td>仅用于 Component 类型，声明该组件可以被蓝图生成</td></tr><tr><td>BlueprintThreadSafe</td><td>仅用于蓝图函数库，声明该类中的函数可在非游戏线程的动画蓝图里调用</td></tr><tr><td>DeprecatedNode</td><td>仅用于 Behaviour Tree Node，声明该节点已被弃用，编译时会有 Warnning</td></tr><tr><td>DeprecationMessage="Message"</td><td>标记弃用的节点，在编译时的警告信息</td></tr><tr><td>DisplayName="Display Name"</td><td>声明该类在蓝图里显示的类型名称</td></tr><tr><td>ShortToolTip="Short tooltip"</td><td>声明在某些情况下（例如在选择父类的下拉框中），用来替代 ToolTip 的显示</td></tr><tr><td>ToolTip="Hand-written tooltip"</td><td>声明自定义的 ToolTip</td></tr></tbody></table>



## 二、常用 Property Specifiers

### 2.1 Property Specifier

<table><thead><tr><th width="271">Property Specifier</th><th>作用</th></tr></thead><tbody><tr><td>AdvancedDisplay</td><td>该属性会被放到 Advanced 分类里</td></tr><tr><td>AssetRegistrySearchable</td><td>该属性和值会被自动注册到 Asset Registry</td></tr><tr><td>BlueprintAssignable</td><td>仅用于多播委托，声明该委托可在蓝图里绑定</td></tr><tr><td>BlueprintAuthorityOnly</td><td>仅用于多播委托，// TODO: 待补充</td></tr><tr><td>BlueprintCallable</td><td>仅用于多播委托，声明该委托可在蓝图里调用</td></tr><tr><td>BlueprintGetter=GetterFunc</td><td></td></tr><tr><td>BlueprintReadOnly</td><td></td></tr><tr><td>BlueprintReadWrite</td><td></td></tr><tr><td>BlueprintSetter=SetterFunc</td><td></td></tr><tr><td></td><td></td></tr></tbody></table>

