#Си_Шарп 

# Анатомия файла XAML
![[Pasted image 20241215130334.png]]
Первая пара файлов — _App.xaml_ , файл XAML, и _App.xaml.cs , файл_ _кода_ C# , связанный с файлом XAML. Оба _файла App.xaml_ и _App.xaml.cs_ вносят вклад в класс с именем App, который является производным от [Application](https://learn.microsoft.com/ru-ru/dotnet/api/system.windows.application?view=windowsdesktop-8.0) . 

Вторая пара файлов — _AppShell.xaml_ и _AppShell.xaml.cs_ , которые вносят вклад в класс с именем `AppShell`, который является производным от [Shell](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.shell) . 

Большинство других классов с файлами XAML вносят вклад в класс, который является производным от [ContentPage](https://learn.microsoft.com/en-us/dotnet/api/microsoft.maui.controls.contentpage) , и определяют пользовательский интерфейс страницы. Это справедливо для файлов _MainPage.xaml_ и _MainPage.xaml.cs_ .

[[Макеты расстановки информации]]