#Lab 4: �������� ������ � ������

## ������ IoT Hub � ������

![Get Access Key](../images/IoTHub_AccessKeys.PNG)

## �������� ��� ��� ������ � IoT-�����

����� ������� ��������� [Getting Started](https://azure.microsoft.com/ru-ru/develop/iot/get-started/) ���� � MSDN. 
��������� ��� ����������, ���� ���������������� � �.�. - � ��������� �������� ����.

� ����� ������ �������� Raspberry Pi 2 -> Windows -> C#. ��� ���������� ���� � ������, ���������� ����� �������� ������ �� NuGet-�����
`Microsoft.Azure.Devices.Client`.



## ����������� Stream Analytics

��� ������, �������� Stream Analytics ��� �������� ������ �� IoT Hub � PowerBI. ������ ����� ����������� ���������� ������ �� ����������� �� �����-�� �������� �������,
��������, 5 ������.

��� ����� ���������� ����� ������:

```
SELECT
    Id, AVG(Temperature) as Temp, [Table], No, 
    MAX(Time) as EndTime, MIN(Time) as BeginTime
INTO [OutBI]
FROM [InHub] TIMESTAMP BY Time
GROUP BY Id,[Table],No,TumblingWindow(Duration(second,5))
```

��� ����� ���������� [���������� �������� � ������� �������� ��������](https://azure.microsoft.com/en-us/documentation/articles/stream-analytics-stream-analytics-query-patterns/).




