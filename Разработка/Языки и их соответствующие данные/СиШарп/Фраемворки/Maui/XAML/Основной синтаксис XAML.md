#Си_Шарп 
# Элементы собственности
В XAML-интерфейсе многоплатформенного приложения .NET (.NET MAUI) свойства классов обычно задаются как атрибуты XML:
```
	<Label Text="Hello, XAML!"
	       VerticalOptions="Center"
	       FontAttributes="Bold"
	       FontSize="18"
	       TextColor="Aqua" />
```
Так же существует альтернативный способ задать свойство в XAML:
```
	<Label Text="Hello, XAML!"
	       VerticalOptions="Center"
	       FontAttributes="Bold"
	       FontSize="18">
	       
	    <Label.TextColor>
	        Aqua
	    </Label.TextColor>
	    
	</Label>
```
- `Text`, `VerticalOptions`, `FontAttributes`и `FontSize`являются _атрибутами свойств_ . Это свойства .NET MAUI, выраженные как атрибуты XML.

- Во втором примере `TextColor`has become a _property element_ . Это свойство .NET MAUI, выраженное как элемент XML.

Синтаксис элемента свойства также может использоваться для нескольких свойств объекта:
```
	<Label Text="Hello, XAML!"
	       VerticalOptions="Center">
	    <Label.FontAttributes>
	        Bold
	    </Label.FontAttributes>
	    <Label.FontSize>
	        Large
	    </Label.FontSize>
	    <Label.TextColor>
	        Aqua
	    </Label.TextColor>
	</Label>
```

