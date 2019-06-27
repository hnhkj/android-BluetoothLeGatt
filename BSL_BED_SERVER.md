# BSL_BED_SERVER 测试代码修改

## 设备端UUID配置

* 0000abf0-0000-1000-8000-00805f9b34fb - Server UUID
	* 0000abf1-0000-1000-8000-00805f9b34fb - 发送数据到设备端
		* SPP_DATA_RECV_CHAR
		* READ&WRITE_NR
	* 0000abf2-0000-1000-8000-00805f9b34fb - 从设备端接收数据
		* SPP_DATA_NOTIFY_CHAR
		* READ&NOTIFY
	* 0000abf3-0000-1000-8000-00805f9b34fb - 发送命令到设备端
		* SPP_COMMAND_CHAR
		* READ&WRITE_NR
	* 0000abf4-0000-1000-8000-00805f9b34fb - 从设备端读取命令
		* SPP_STATUS_CHAR
		* READ & NOTIFY
	* 0000abf5-0000-1000-8000-00805f9b34fb - 保留
		* SPP_HEARTBEAT_CHAR
		* READ&WRITE_NR&NOTIFY

## 修改内容

#### AndroidManifest.xml

蓝牙扫描添加对`Android 60`以后的设备支持，在5.0之前，`Android`默认授予了位置权限，在之后的版本需要对`App`进行授权才可以使用蓝牙扫描代码。

```
    <!-- Needed only if your app targets Android 5.0 (API level 21) or higher. -->
    <uses-permission android:name="android.hardware.location.gps" />
    <!-- Need if your app targets Android 6.0 or higher -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

#### SampleGattAttributes.java

* 添加控制器UUID

```java
public class SampleGattAttributes {
    public static String BSL_BED_SERVER = "0000abf0-0000-1000-8000-00805f9b34fb";
    public static String BED_DATA_RECV_CHAR = "0000abf1-0000-1000-8000-00805f9b34fb";
    public static String BED_DATA_NOTIFY_CHAR = "0000abf2-0000-1000-8000-00805f9b34fb";
    public static String BED_COMMAND_CHAR = "0000abf3-0000-1000-8000-00805f9b34fb";
    public static String BED_STATUS_CHAR = "0000abf4-0000-1000-8000-00805f9b34fb";
    static {
        // Bed Services
        attributes.put(BSL_BED_SERVER, "BSL Bed Server");
        // Bed Characteristics
        attributes.put(BED_DATA_RECV_CHAR, "Bed Data Write");
        attributes.put(BED_DATA_NOTIFY_CHAR, "Bed Data Notify");
        attributes.put(BED_COMMAND_CHAR, "Bed Command");
        attributes.put(SPP_STATUS_CHAR, "Bed Status");
    }
```

#### BluetoothLeService.java

* 添加`UUID_BED_DATA_NOTIFY`

```java
    public final static UUID UUID_BED_DATA_NOTIFY =
            UUID.fromString(SampleGattAttributes.BED_DATA_NOTIFY_CHAR);
```

* 使能`UUID_BED_DATA_NOTIFY`通知

```java
public void setCharacteristicNotification(BluetoothGattCharacteristic characteristic,
                                              boolean enabled) {
        ......
        // This is specific to Heart Rate Measurement.
        if (UUID_HEART_RATE_MEASUREMENT.equals(characteristic.getUuid())) {
            ......
        }
        }else if (UUID_BED_DATA_NOTIFY.equals(characteristic.getUuid())) {
            BluetoothGattDescriptor descriptor = characteristic.getDescriptor(
                    UUID.fromString(SampleGattAttributes.CLIENT_CHARACTERISTIC_CONFIG));
            descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
            mBluetoothGatt.writeDescriptor(descriptor);
        }
```

## 应用

#### 设备列表子项响应

当点特征列表向后，`DeviceControlActivity.java`中的`servicesListClickListner()`函数对选择的项目进行
响应。

函数流程：

* 根据选择相应的项目，获取`Characteristics`信息
* 根据`Characteristics`信息，提取`Properties`信息。该信息包含了`Characteristics`的权限信息。
* 更具获得的权限，通过`setCharacteristicNotification()`函数对`Notify`进行使能
* 设备端可以发送数据到Android。

权限信息定义相见`BluetooothGattCharcteristic.java`文件。

```java
private final ExpandableListView.OnChildClickListener servicesListClickListner =
            new ExpandableListView.OnChildClickListener() {
                @Override
                public boolean onChildClick(ExpandableListView parent, View v, int groupPosition,
                                            int childPosition, long id) {
                    if (mGattCharacteristics != null) {
                        final BluetoothGattCharacteristic characteristic =
                                mGattCharacteristics.get(groupPosition).get(childPosition);
                        final int charaProp = characteristic.getProperties();
                        if ((charaProp | BluetoothGattCharacteristic.PROPERTY_READ) > 0) {
                            // If there is an active notification on a characteristic, clear
                            // it first so it doesn't update the data field on the user interface.
                            if (mNotifyCharacteristic != null) {
                                mBluetoothLeService.setCharacteristicNotification(
                                        mNotifyCharacteristic, false);
                                mNotifyCharacteristic = null;
                            }
                            mBluetoothLeService.readCharacteristic(characteristic);
                        }
                        if ((charaProp | BluetoothGattCharacteristic.PROPERTY_NOTIFY) > 0) {
                            mNotifyCharacteristic = characteristic;
                            mBluetoothLeService.setCharacteristicNotification(
                                    characteristic, true);
                        }
                        return true;
                    }
                    return false;
                }
    };
```

#### 获取控制器数据

控制器发送的数据由函数 `onCharacteristicChanged()`接收后，通过`broadcastUpdate()`函数对接收
的数据进行分析处理。最后进行呈现。

onCharacteristicChanged()函数

```java
        @Override
        public void onCharacteristicChanged(BluetoothGatt gatt,
                                            BluetoothGattCharacteristic characteristic) {
            broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
        }
```

broadcastUpdate()函数

```java
    private void broadcastUpdate(final String action,
                                 final BluetoothGattCharacteristic characteristic)
```

## 参考

UUID列表

<https://www.cnblogs.com/bulazhang/p/8450172.html>