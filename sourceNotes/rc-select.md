版本：8.8.4 （项目当前使用）

repo：<https://github.com/react-component/select/tree/8.8.4/src>

marker：combobox模式及antuComplete组件

------

## Select.tsx -- Select类

### 静态方法

#### getOptionsFromChildren

传入option子元素数组，如果是optGroup，把里面的option子元素拿出来，返回所有的option子元素。

#### getLabelFromOptions

参数：props: Select的props - 必传， option: any - 必传

功能：返回option的label属性值，label属性key为props.optionLabelProp

#### getOptionsInfoFromProps

参数：prop: Select的props - 必传，preState: Select的state - 可不传

功能：返回optionsInfo对象

逻辑：

```js
public static getOptionsInfoFromProps = (props, preState) => {
    const options = Select.getOptionsFromChildren(props.children); // 获取所有的		option子元素
	const optionsInfo = {};
	options.forEach(option => {
   		const singleValue = getValuePropValue(option); // 获取option的value属性值
    	optionsInfo[getMapKey(singleValue)] = {
            option,
            value: singleValue,
            label: Select.getLabelFromOption(props, option),
            title: option.props.title,
            disabled: option.props.disabled
    	}; // optionsInfo的键的格式为‘value值类型-value值：number-1’, 每个键值是一个			对象，其均具有option,value,label,title,disabled五个属性
	})；
    if (preState) {
        // 保留preState中的原有optionsInfo
        const oldOptionsInfo = preState.optionsInfo;
        const value = preState.value;
        if (value) {
            value.forEach(v => {
                const key = getMapKey(v);
                if (!optionsInfo[key] && oldOptionsInfo[key] !== undefined) {
                    optionsInfo[key] = oldOptionsInfo[key];
                }
            });
        }
    }
	return optionsInfo;
}
```

#### getValueFromProps

参数：props: Select的props - 必传， useDefaultValue：boolean - 可不传 - 是否使用默认值

功能：从传入的props中获取value值（数组形式）

逻辑：

1. 确定value数组：props中存在value属性并且不使用默认值时，取props.value数组化；props存在defaultValue属性并且使用默认值时，取props.defaultValue数组化；
2. 考虑props.labelInValue为true的情况，value数组的各项分别为1中得到的各项的key属性；

#### getInputValueForCombobox

参数：props：Select的props - 必传，optionsInfo: any - 必传，useDefaultValue：boolean - 可不传 - 是否使用默认值

功能：针对combobox模式，获取输入值：空字符串|value值|

逻辑：

1. 定义value变量为空数组，若props中存在value属性并且不使用默认值时，取props.value数组化赋给value变量；若props存在defaultValue属性并且使用默认值时，取props.defaultValue数组化赋给value变量；
2. value数组是空数组的话，直接返回空字符串；否则取数组第一项赋给变量value;
3. 定义label变量为value值；
4. props.labelInValue为true时，label = value.label，否则optionsInfo参数中存在value对应的键，则取该键值的label属性值
5. 返回label变量值；

#### getDerivedStateFromProps生命周期函数

功能：设置新的state。

逻辑：

```js
newState = {
    optionsInfo: prevState.skipBuildOptionsInfo为true时为prevState.optionsInfo,否则为nextProps的optionsInfo,
    skipBuildOptionsInfo: false,
    nextProps存在open属性时=> open: nextProps.open,
    nextProps存在value属性时 => value: nextProps的value属性值，
    nextProps存在value属性并且combobox模式时=> inputValue: 输入值
}
```



## util工具方法

#### getValuePropValue

参数：child: any - 必传

功能：返回child的value属性值

逻辑：

```js
const props = child.props;
if ('value' in props) return props.value;
if (child.key) return child.key;
if (child.type && child.type.isSelectOptGrop && child.lable) return props.lable;
```

#### getPropValue

参数：child: Option - 必传，prop - 可不传

功能：返回child的prop值（prop为value时，返回getValuePropValue方法获取的value属性值）

#### toArray

参数：value：valueType | undefined - 必传

功能：返回value的数组形式；

exmaple: toArray(undefined) => []; toArray(1) => [1]; toArray([1, 2]) => [1, 2];

#### getMapKey

参数：value必传

功能：返回字符串：value类型-value

example：getMapKey(1) => 'number-1'

