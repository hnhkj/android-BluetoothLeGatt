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

## 参考

UUID列表

<https://www.cnblogs.com/bulazhang/p/8450172.html>