#Lab 4: �������� ������ � ������

## ������ IoT Hub � ������

��� ������ ������� � ������ ���� IoT Hub (������ "�������� �����"). ����� �������� ���� ���������� ������ �����������:

![Get Access Key](../images/IoTHub_AccessKeys.PNG)

## ������������ � IoT-���� � ������� ������ ����������� ��� ����������

����������� � ���������� [Device Explorer](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md). � ��� �������
������ ����������� � IoT-����:

![Device Explorer 1](../images/DeviceExplorer1.PNG)

����� ����� ��������� �� ������� "Management" � �������� ����� ����������. ����� ������ ������� ������� �� ������ � ����������� � �������� "Copy Connection String".

![Device Explorer 2](../images/DeviceExplorer2.PNG)

## �������� ��� ��� ������ � IoT-�����

����� ������� ��������� [Getting Started](https://azure.microsoft.com/ru-ru/develop/iot/get-started/) ���� � MSDN. 
��������� ��� ����������, ���� ���������������� � �.�. - � ��������� �������� ����.

� ����� ������ �������� Raspberry Pi 2 -> Windows -> C#. ��� ���������� ���� � ������, ���������� ����� �������� ������ �� NuGet-�����
`Microsoft.Azure.Devices.Client`. � ���������� ���� �� �������� ��������� ������ ����������� �� ��, ������� ���� �������� �� ���������� ����.

### �������� ������ � IoT Hub

��� �������� ������ ������������ ��������� ���:

```
iothub = DeviceClient.CreateFromConnectionString(DeviceConnectionString);
await iothub.OpenAsync();
...
var d = new TempData("RPi", t, Table, No);
var s = Newtonsoft.Json.JsonConvert.SerializeObject(d);
var b = Encoding.UTF8.GetBytes(s);
await iothub.SendEventAsync(new Message(b));
```
����� ������������ ���������� `Newtonsoft.Json` (������� ���� ���������� ����� Nuget) � ��� ��� ������������� ������ �� ����� ������������� ���������:

```
    public class TempData
    {
        public string Id { get; set; }
        public double Temperature { get; set; }
        public DateTime Time { get; set; }
        public int Table { get; set; }
        public int No { get; set; }
        public TempData(string id, double Temp, int Table, int No)
        {
            Time = DateTime.Now;
            Temperature = Temp;
            Id = id;
            this.Table = Table;
            this.No = No;
        }
    }
```

### ��������� ������ �� IoT Hub
��� ����� ��������� �� IoT Hub ����� ������������ ��������� ���:
```
        private async Task Receive()
        {
            while (true)
            {
                var msg = await iothub.ReceiveAsync();
                if (msg != null)
                {
                    var s = Encoding.ASCII.GetString(msg.GetBytes());
                    // ������� ���-�� � ���������� ����������, ��������, ������ ���������
                    LED.SetInt(int.Parse(s));
                    await iothub.CompleteAsync(msg);
                }
            }
        }
```

## ����������� Stream Analytics

��� �������� ������ �� IoT Hub � ������� �������� ����� ������������ Stream Analytics.

��� ������, �������� Stream Analytics ��� �������� ������ �� IoT Hub � PowerBI. ������ ����� ����������� ���������� ������ �� ����������� �� �����-�� �������� �������,
��������, 5 ������.

��� ����� ���������� ������� ������ Stream Analytics, ���������������� � ��� ������� � �������� ������ ������, � ������ ������. 

� �������� ������� ������ ���������� IoT Hub, ������� ������� ������ `InHub`. 
� �������� �������� ������ - Power BI, ������ �� `OutBI`. 
�������� ��������, ��� � ������� ������ ������� Azure ��� ���������������� PowerBI ���������� ������������ ������ ������ http://manage.windowsazure.com.

��� ���������� ������ �� 5 ������ ���������� ����� ������:

```
SELECT
    Id, AVG(Temperature) as Temp, [Table], No, 
    MAX(Time) as EndTime, MIN(Time) as BeginTime
INTO [OutBI]
FROM [InHub] TIMESTAMP BY Time
GROUP BY Id,[Table],No,TumblingWindow(Duration(second,5))
```

��� ����� ���������� [���������� �������� � ������� �������� ��������](https://azure.microsoft.com/en-us/documentation/articles/stream-analytics-stream-analytics-query-patterns/).

����� ���������������� ������� Stream Analytics ���������� ��������� ������� �� ����������.

## ����������� ����� � PowerBI

����� ����, ��� Stream Analytics ����� ��������, �� ������ ������� � ������ PowerBI ��������� ������:

![PowerBI](../images/PowerBI.PNG)

## ���������� ������ � ������ �����������
������ �������� ������������� Stream Analytics - �������������� � ������������ ���������. ������� ��� ���� �������� �������� ������ � �����������
`OutAlert`, ����������� �� Event Hub �� ���������� �����������:

 * ������������ ��� ��������� ����: DevConHub-ns
 * ��� ������������� �������: devconhub
 * ��� �������� ������������� �������: RootManageSharedAccessKey
 * ���� �������� ������������� �������: QEZ4Gvc+QXhr4DiVpGf2XSVL1Yo7bc/g+aqK3uVrfjg=

����� ������� � ����� ������� ��������� ������ (��� ����� ����������� ������������� ������� Stream Analytics � ����� ����� ��� ���������):

```
SELECT Id, Temperature, Time, [Table], No
INTO [OutAlert]
FROM [InHub] TIMESTAMP BY Time
WHERE Temperature>30
```

������ ��� ��������������� ������� ��������� � ������ ������� EventHub � ����� ������������ �� ������ �����������, ������� ��������� ����������.
