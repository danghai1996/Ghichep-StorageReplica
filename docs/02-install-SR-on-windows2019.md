# Cài đặt và cấu hình Storage Replica trên Windows 2019 server Datacenter Edition

# Yêu cầu: 
2 máy cài đặt sẵn HĐH Windows Server 2019 Datacenter Edition

- WIN-PC01: 192.168.10.37/24 - 4 Cores - 4 GB RAM
- WIN-PC02: 192.168.10.30/24 - 4 Cores - 4 GB RAM

Mỗi máy có 2 phân vùng giống hệt nhau. 1 là nơi Relicate dữ liệu, 2 là nơi lưu Log. 2 ổ sẽ sử dụng Format là GPT và định dạng File System là **ReFS** thay vì **NTFS**.

Máy 1:

<img src = "../images/Screenshot_11.png">

Máy 2:

<img src = "../images/Screenshot_11.png">

# Cấu hình:
## Cấu hình Active Directory Domain
2 máy cần ở chung 1 domain. 

Trong bài lab này, ta set 1 máy làm Domain Controller, máy còn lại sẽ join domain của máy 1.

### Cấu hình máy win-PC01 làm controller
<img src = "../images/Screenshot_13.png">

<img src = "../images/Screenshot_14.png">

<img src = "../images/Screenshot_15.png">

<img src = "../images/Screenshot_16.png">

<img src = "../images/Screenshot_17.png">

<img src = "../images/Screenshot_18.png">

<img src = "../images/Screenshot_19.png">

<img src = "../images/Screenshot_20.png">

<img src = "../images/Screenshot_21.png">

<img src = "../images/Screenshot_22.png">

<img src = "../images/Screenshot_23.png">

<img src = "../images/Screenshot_24.png">

<img src = "../images/Screenshot_25.png">

Truy cập lại bằng tài khoản `Administrator@vnptcloud.local`., Mật khẩu: pass đã đặt ở trên

Sau khi truy cập, ta sẽ thấy máy đã Join Domain.

<img src = "../images/Screenshot_26.png">


### Tiến hành Join Domain trên máy WIN-PC02:
Chuyển DNS thành IP của máy controller:

<img src = "../images/Screenshot_30.png">

Cấu hình join domain

<img src = "../images/Screenshot_27.png">

<img src = "../images/Screenshot_28.png">

<img src = "../images/Screenshot_29.png">

<img src = "../images/Screenshot_31.png">

<img src = "../images/Screenshot_32.png">

<img src = "../images/Screenshot_33.png">

<img src = "../images/Screenshot_34.png">

<img src = "../images/Screenshot_35.png">

Đợi restart và truy cập bằng user Truy cập lại bằng tài khoản `Administrator@vnptcloud.local`., Mật khẩu: pass đã đặt ở trên

<img src = "../images/Screenshot_36.png">

Quay lại máy win-PC01, Mở **“Active Directory Users and Computers”** để kiểm tra:

<img src = "../images/Screenshot_37.png">

Ta đã thấy máy vừa join domain.
<img src = "../images/Screenshot_38.png">

## Cài đặt tính năng Storage Replication trên cả 2 máy
Enabled Storage Replica:

<img src = "../images/Screenshot_5.png">

<img src = "../images/Screenshot_6.png">

<img src = "../images/Screenshot_7.png">

<img src = "../images/Screenshot_8.png">

<img src = "../images/Screenshot_9.png">

Sau khi quá trình hoàn tất ta sẽ thấy có thông báo yêu cầu Restart OS:

<img src = "../images/Screenshot_10.png">

Restart OS.

## Chạy Test:
Chạy lệnh Test để kiểm tra điều kiện 2 máy có đáp ứng không
```powershell
Test-SRTopology -SourceComputerName “WIN-PC01” -SourceVolumeName “r:” -SourceLogVolumeName “l:” -DestinationComputerName “WIN-PC02” -DestinationVolumeName “r:” -DestinationLogVolumeName “l:” -DurationInMinutes 10 -ResultPath c:\Temp
```

Đợi quá trình thực hiện xong. Mở file report xem kết quả:

<img src = "../images/Screenshot_76.png">

## Create replication partnership
Tạo mối quan hệ giữa 2 máy. 

Trên máy WIN-PC01, khởi chạy PowerShell bằng quyền Administrator. Chạy đoạn lệnh sau:

```powershell
New-SRPartnership -SourceComputerName "WIN-PC01" -SourceRGName rg01 -SourceVolumeName "R:" -SourceLogVolumeName "L:" -DestinationComputerName "WIN-PC02" -DestinationRGName rg02 -DestinationVolumeName "R:" -DestinationLogVolumeName "L:" -LogSizeInBytes 1gb
```

**Trong đó:**
- `-SourceComputerName WIN-PC01` : Tên máy cần replicate dữ liệu (`WIN-PC01`)
- `-SourceRGName Replication01` : Đặt tên cho Replica Group của máy nguồn (`rg01`)
- `-SourceVolumeName R:` : Ổ lưu dữ liệu cần replicate trên máy nguồn (`R:\`)
- `-SourceLogVolumeName L:` : Ổ lưu log trên máy nguồn WIN-PC01 (`L:\`)
- `-DestinationComputerName WIN-PC02`: Tên máy nhận replicate dữ liệu (`WIN-PC02`)
- `-DestinationRGName Replication02` : Đặt tên cho Replica Group của máy nhận WIN-PC02(`rg02`)
- `-DestinationVolumeName R:` : Ổ lưu dữ liệu cần replicate trên máy nhận WIN-PC02(`R:\`)
- `-DestinationLogVolumeName L:` : Ổ lưu log trên máy nhận WIN-PC02(`L:\`)

Output:
```
DestinationComputerName : WIN-PC02
DestinationRGName       : rg02
Id                      : 5c3249a1-b3ef-4bca-a804-92ba4ea46a35
SourceComputerName      : WIN-PC01
SourceRGName            : rg01
PSComputerName          :
```

<img src = "../images/Screenshot_39.png">

Sau khi cấu hình lần đầu, ta kiểm tra staus sẽ thấy ở trạng thái `InitialBlockCopy`. Sau khi quá trình replication ban đầu hoàn tất, trạng thái sao chép sẽ là `ContinuouslyReplicating`.

<img src = "../images/Screenshot_50.png">

<img src = "../images/Screenshot_51.png">

# Kiểm tra:
### Thêm Performance Monitor cho Storage Replica: Thực hiện trên cả 2 máy
Ta sẽ thực hiện thêm giám sát cho tính năng. Khởi chạy PowerShell, gõ lệnh: `PerfMon.msc` để Mở Performance Monitor.

- Chọn Performance Monitor -> Chọn Add:

    <img src = "../images/Screenshot_41.png">

- Tìm đến Storage Replica Statistics -> Add -> OK

    <img src = "../images/Screenshot_42.png">

- Chuyển về dạng Report:

    <img src = "../images/Screenshot_48.png">

- Khi không có dữ liệu đồng bộ, ta sẽ thấy các giá trị bằng 0:

    <img src = "../images/Screenshot_43.png">

- Thử thêm dữ liệu vào ổ Replica trên máy WIN-PC01, ta sẽ thấy các giá trị thay đổi, điều này cho biết việc đồng bộ đang được thực hiện. Khi thực hiện xong thì các giá trị lại trở về 0:

    <img src = "../images/Screenshot_44.png">

### Thử đổi ngược lại chiều đồng bộ kiểm tra dữ liệu trên WIN-PC02
Trên máy WIN-PC01, dữ liệu trên ổ Replica

<img src = "../images/Screenshot_40.png">


Trên máy WIN-PC02, chạy lệnh để đổi chiều.
```PowerShell
Set-SRPartnership -NewSourceComputerName WIN-PC02 -SourceRGName rg02 -DestinationComputerName WIN-PC01 -DestinationRGName rg01
```

Nhập `Y` để đồng ý:

<img src = "../images/Screenshot_45.png">

Sau đó, kiểm tra trên máy 2 sẽ thấy ổ Replica đã có thể truy cập. Dữ liệu trong ổ đầy đủ:

<img src = "../images/Screenshot_46.png">

<img src = "../images/Screenshot_47.png">

Trên máy WIN-PC01 sẽ trở thành máy Destination. Ổ Replica không còn truy cập được nữa:

<img src = "../images/Screenshot_49.png">

## Check Event Log của Storage Replica
```
Get-WinEvent -ProviderName Microsoft-Windows-StorageReplica -Max 40
```

<img src = "../images/Screenshot_52.png">

## Disable Replication and Remove Partnership
Chạy lần lượt 2 lệnh sau trên PowerShell:
```
Get-SRPartnership | Remove-SRPartnership

Get-SRGroup | Remove-SRGroup
```

# Tham khảo:
- https://docs.microsoft.com/en-us/windows-server/storage/storage-replica/server-to-server-storage-replication
- https://nedimmehic.org/2017/02/22/configure-storage-replication-server-to-server/
- https://robertsmit.wordpress.com/2018/10/30/step-by-step-server-to-server-storage-replication-with-windows-server-2019-storage-replica-windowsadmincenter-storagereplica-windowsserver2019-refs-sr-azure/