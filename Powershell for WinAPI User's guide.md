# Содержание
- [Об этом руководстве](#Об-этом-руководстве)
- [Источники для самостоятельного поиска информации](#Источники-для-самостоятельного-поиска-информации)
- [Подключение к Windows API](#Подключение-к-Windows-API) 
- [Способ 1](#Способ-1) 
- [Способ 2](#Способ-2) 
- [Некоторые примеры использования](#Практические-примеры-использования)

# Об этом руководстве
Этот документ представляет собой руководство пользователя по взаимодействию с Windows API 
c использованием PowerShell. 
Для успешного использования нужно быть знакомым с операционной системой Windows, 
владеть основными приемами работы в ней. Преимуществом будет являться знание языка программирования C++ или C#. 

Руководство предназначено для следующих целей:
- Помочь вызывать нативный Windows API с помощью PowerShell для выполнения конкретных задач;
- Показать некоторые примеры использования.

# Источники для самостоятельного поиска информации
Вы можете использовать следующие источники для самостоятельного поиска информации:
### Windows API
- [страница на сайте Microsoft](https://learn.microsoft.com/ru-ru/windows/win32/);
- [страница на GitHub](https://github.com/MicrosoftDocs/win32/blob/docs/desktop-src/apiindex/windows-api-list.md);
### PowerShell
- [страница на сайте Microsoft](https://learn.microsoft.com/ru-ru/powershell/);
- [страница на GitHub](https://github.com/PowerShell/PowerShell)

# Подключение к Windows API
Существует два основных метода доступа к Windows API из PowerShell:
- Использование командлета Add-Type для компиляции кода C#. *Данный метод является официально задокументированным на сайте Microsoft*. [По этой ссылке можно ознакомиться подробнее c командлетом Add-Type](https://learn.microsoft.com/ru-RU/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.3)
- Использование ссылки на закрытый тип в .NET Framework, который вызывает метод.
Ниже представлено подробное описание, как можно использовать каждый метод для получения доступа к Windows API.
## Способ 1
**Add-Type** - это команда в PowerShell, которая позволяет загружать и компилировать .NET-сборки в текущую сессию PowerShell. Это может быть полезно, если вы хотите использовать функциональность, которая не доступна в стандартных модулях PowerShell. Например, существует метод **CopyFile** в библиотеке kernel32.dll, который предоставляет возможность взаимодействовать с файлами, созданными службами теневого копирования томов. Для ознакомления с синтаксисом, пройдите по [ссылке](https://learn.microsoft.com/ru-ru/windows/win32/api/winbase/nf-winbase-copyfile?redirectedfrom=MSDN). В данном примере, синтаксис представлен на языке С++. Ниже в таблице представлено сопоставление типов данных на С++ и С#.

| **С++**  | **С#** |
| -------- | -------|
|BOOL|bool |
|LPCTSTR|string|

Ознакомившись с синтаксисом и с эквивалентами данных, необходимо открыть PowerShell и выполнить скрипт. Если по каким-то причинам у вас не установлен PowerShell, то необходимо следуя [инструкции](https://learn.microsoft.com/ru-ru/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.3) в базе знаний Microsoft произвести установку.

Запуск PowerShell возможен как в панели "Пуск",
![image](https://github.com/BondarevDima/Public_repository/assets/126958591/2ea09866-5e52-4ed9-b564-b80b026d62b4)

так и нажатием сочетания клавиш Win+R, где в появившимся диалоговом окне необходимо ввести **powershell**.
![image](https://github.com/BondarevDima/Public_repository/assets/126958591/e618df0a-5ea7-4c35-b52b-0b6700bfbae1)


Исходный код представлен ниже:
```
$MethodDefinition = @'
[DllImport("kernel32.dll", CharSet = CharSet.Unicode)]
public static extern bool CopyFile(string lpExistingFileName, string lpNewFileName, bool bFailIfExists);
'@
$Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -Namespace 'Win32' -PassThru
$Kernel32::CopyFile("$($Env:SystemRoot)\System32\calc.exe", "$($Env:USERPROFILE)\Desktop\calc.exe", $False)
```
```$MethodDefinition``` Переменная содержит определение метода С#, которое соответствует сигнатуре C ++ ```CopyFile```.
Атрибут ```DllImport``` указывает, что ```CopyFile``` метод предоставляется в ```kernel32.dll``` библиотеке в качестве статической точки входа.

## Способ 2
В случае, если способ с использованием командлета Add-Type Вам не подходит, то возможно воспользоваться способом ссылки на закрытый тип в .NET, 
который вызовет метод. Данный способ будет полезен для пользователей, кому важно при выполнении скрипта отсутствие записи временных файлов на жёсткий диск.
Подходит опытным пользователям, т.к. требует более глубоких знаний .NET Framework, 
Существует репозиторий [Script Center](http://gallery.technet.microsoft.com/scriptcenter/Find-WinAPIFunction-4166b223), где описана вспомогательная функция Find-WinAPIFunction, которая будет необходима для дальнейшей работы. Используя её и функцию **Copy-RawItem**, возможно выполнить те же действия, которые были описаны в способе 1.
Исходный код представлен ниже:
```
function Copy-RawItem {
    [CmdletBinding()]
    [OutputType([System.IO.FileSystemInfo])]
    Param (
        [Parameter(Mandatory = $True, Position = 0)]
        [ValidateNotNullOrEmpty()]
        [String]
        $Path,
        [Parameter(Mandatory = $True, Position = 1)]
        [ValidateNotNullOrEmpty()]
        [String]
        $Destination,
        [Switch]
        $FailIfExists
    )
    $mscorlib = [AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { 
        $_.Location -and ($_.Location.Split('\')[-1] -eq 'mscorlib.dll')
    }
    $Win32Native = $mscorlib.GetType('Microsoft.Win32.Win32Native')
    $CopyFileMethod = $Win32Native.GetMethod(
        'CopyFile', ([Reflection.BindingFlags] 'NonPublic, Static'))
    $CopyResult = $CopyFileMethod.Invoke(
        $null, @($Path, $Destination, ([Bool] $PSBoundParameters['FailIfExists'])))
    $HResult = [System.Runtime.InteropServices.Marshal]::GetLastWin32Error()
    if ($CopyResult -eq $False -and $HResult -ne 0) {
        throw New-Object ComponentModel.Win32Exception($HResult)
    }
    else {
        Write-Output(Get-ChildItem $Destination)
    }
}
```
# Практические примеры использования
One of the simplest tasks where you can visually demonstrate the capabilities of the API is to create a user notification window. This task is easy for beginners, since little data is required to write the script. Using the [Pinvoke*](https://www.pinvoke.net/) and finding a C++ data structure record in MSDN, we will use the C# equivalent of the record.

*The PInvoke.net wiki is a great resource for information on common Windows APIs and how to call them.
```
Add-Type -TypeDefinition @"
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
 
public static class User32
{
    [DllImport("user32.dll", CharSet=CharSet.Auto)]
        public static extern bool MessageBox(
            IntPtr hWnd,     /// Parent window handle 
            String text,     /// Text message to display
            String caption,  /// Window caption
            int options);    /// MessageBox type
}
"@
 
[User32]::MessageBox(0,"Text","Caption",0x4)
```
Example:
![Example](https://github.com/BondarevDima/Public_repository/assets/126958591/976a57a3-6d61-447b-8adf-71561b653857)
