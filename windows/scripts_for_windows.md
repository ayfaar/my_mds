# Скрипты на разные случаи

## VBS

## IS PROCESS RUNNING?(ЗАПУЩЕН ЛИ ПРОЦЕСC?)

Функция *IsProcessRunning* возвращает True, если в выборке процессов существует хотябы один с нашим именем.

```vbscript
Function IsProcessRunning( processNameExe )
   IsProcessRunning = GetObject("WinMgmts:\\.\root\cimv2") _
   .ExecQuery("SELECT * FROM Win32_Process WHERE Name LIKE '" & processNameExe & "'") _
      .Count > 0
End Function
```

Использование функции IsProcessRunning

```vbscript
Function IsProcessRunning( processNameExe )
   IsProcessRunning = GetObject("WinMgmts:\\.\root\cimv2") _
   .ExecQuery("SELECT * FROM Win32_Process WHERE Name LIKE '" & processNameExe & "'") _
      .Count > 0
End Function

'Использование
If IsProcessRunning("notepad.exe") Then
  Msgbox "notepad process is running"
Else
  Msgbox "notepad process is not running"
End If

' Данный код выводит окно сообщение о том запущен ли "notepad.exe"
MsgBox IsProcessRunning("notepad.exe")
```

### RUN APP(запуск приложения)

Запуск firefox и перевод его в полноэкранный режим, нажатием 'F11'

```vbscript
set WshShell = WScript.CreateObject("WScript.Shell")
WshShell.Run "firefox"
WScript.Sleep 5000
WshShell.AppActivate "firefox"
WScript.Sleep 100
WshShell.SendKeys "{F11}"
WScript.Sleep 500
```

## CMD

### CMD Фаервол

Включение|Отключени всех профилей

```dos
netsh advfirewall set allprofiles state off
netsh advfirewall set allprofiles state on
```

Включение профилей по отдельности

```dos
netsh advfirewall set domainprofile state on
netsh advfirewall set privateprofile state on
netsh advfirewall set publicprofile state
```

Показать все правила фаервола по имени

```dos
netsh advfirewall firewall show rule name=all
```

Разрешить принимать запросы от пинга "echo request"

```dos
netsh advfirewall firewall add rule name="ICMP Allow incoming V6 echo request" protocol=icmpv6:8,any dir=in action=allow

rem netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=block
```

Запрет всех входящих соединений и разрешение входящих

```dos
netsh advfirewall set allprofiles firewallpolicy blockinbound,allowoutbound
```

Разрешение входящих протоколов TCP и UDP на 80 порт

```dos
netsh advfirewall firewall add rule name="Open Port 80" dir=in action=allow protocol=TCP localport=80
netsh advfirewall firewall add rule name="Open Port 80" dir=in action=allow protocol=UDP localport=80
```

Запрет входящих протоколов TCP,UDP на 80 порт:

```dos
netsh advfirewall firewall add rule name="HTTP" protocol=TCP localport=80 action=block dir=IN
netsh advfirewall firewall add rule name="HTTP" protocol=UDP localport=80 action=block dir=IN
```

Открыть диапозон портов для исходящего UDP трафика

```dos
netsh advfirewall firewall add rule name="Port range" protocol=UDP localport=5000-5100 action=allow dir=OUT
```

Удалить правило по имени

```dos
netsh advfirewall firewall delete rule name="HTTP"
```

Правило ограничивающие подключение одно ip-адреса

```dos
netsh advfirewall firewall add rule name="HTTP" protocol=TCP localport=80 action=allow dir=IN remoteip=192.168.0.1
```

Ограничение подключений с диапазона ip-адресов или сетей

```dos
netsh advfirewall firewall add rule name="HTTP" protocol=TCP localport=80 action=block dir=IN remoteip=192.168.0.0/24
netsh advfirewall firewall add rule name="HTTP" protocol=TCP localport=80 action=allow dir=IN remoteip=192.168.0.50-192.168.0.70
netsh advfirewall firewall add rule name="HTTP" protocol=TCP localport=80 action=block dir=IN remoteip=localsubnet
```

Разрешить соединения для программы MyApp.exe

```dos
netsh advfirewall firewall add rule name="My Application" dir=in action=allow program="C:\MyApp\MyApp.exe" enable=yes
```

Комбинирование параметров.Мы создали правило, которое разрешает входящие соединения к приложению MyApp из сетей с ip-адресами 157.60.0.1,172.16.0.0/16 и доменным профилем сетевого подключения.

```dos
netsh advfirewall firewall add rule name="My Application" dir=in action=allow program="C:\MyApp\MyApp.exe" enable=yes remoteip=157.60.0.1,172.16.0.0/16,LocalSubnet profile=domain
```

### environment variables

```dos
echo off
SET nvim_bin_dir="%~dp0%nvim-win64\bin"


SETX PATH "%nvim_bin_dir%"

Pause
```

***
