---
title: Flutter TimePicker
date: 2022-03-22 10:17:22
categories:
- TimePicker
- Flutter
tags:
- Flutter
- TimePicker
---

{% asset_img 1.png 示意图 width="400" %}

在本文中，我们将快速探索日期和时间选择器。 
我们不会在这个应用程序中使用任何包。我们将使用 Flutter 中可用的一些核心功能来实现这一点。 我们将在您的 Flutter 应用程序中实现日期和时间选择器的演示。

### 目录 ：
 * 日期和时间选择器
 * 代码实现
 * 代码文件
 * 结论

{% asset_img 2.jpg 示意图 width="400" %}

### 日期和时间选择器
Flutter 的日期和时间选择器，您可以选择英语、荷兰语和任何其他您想要的语言的日期/时间/日期和时间，您还可以自定义您自己的选择器内容。

{% asset_img 3.gif 示意图 width="400" %}

### 代码实现
在 lib 文件夹中创建一个名为 DateTimePicker.dart 的新 dart 文件。

{% asset_img 4.png 示意图 width="400" %}

> 在此屏幕中，您将能够通过在您的应用程序中点击它们来选择日期和时间。

```
Future<Null> _selectDate(BuildContext context) async {
  final DateTime picked = await showDatePicker(
      context: context,
      initialDate: selectedDate,
      initialDatePickerMode: DatePickerMode.day,
      firstDate: DateTime(2015),
      lastDate: DateTime(2101));
  if (picked != null)
    setState(() {
      selectedDate = picked;
      _dateController.text = DateFormat.yMd().format(selectedDate);
    });
}
```

> 初始化 DateTime 选择器类。这将保存我们选择的日期。

{% asset_img 5.png 示意图 width="400" %}

> 在 TextFromField 的 onTap 中，我们调用 _selectDate 函数，然后将显示选择器并保存选择的日期和时间。

```
InkWell(
  onTap: () {
    _selectDate(context);
  },
  child: Container(
    width: _width / 1.7,
    height: _height / 9,
    margin: EdgeInsets.only(top: 30),
    alignment: Alignment.center,
    decoration: BoxDecoration(color: Colors.grey[200]),
    child: TextFormField(
      style: TextStyle(fontSize: 40),
      textAlign: TextAlign.center,
      enabled: false,
      keyboardType: TextInputType.text,
      controller: _dateController,
      onSaved: (String val) {
        _setDate = val;
      },
      decoration: InputDecoration(
          disabledBorder:
              UnderlineInputBorder(borderSide: BorderSide.none),
         contentPadding: EdgeInsets.only(top: 0.0)),
    ),
  ),
),
```

它只是一个 _selectTime 函数，如下所示。

```
Future<Null> _selectTime(BuildContext context) async {
  final TimeOfDay picked = await showTimePicker(
    context: context,
    initialTime: selectedTime,
  );
  if (picked != null)
    setState(() {
      selectedTime = picked;
      _hour = selectedTime.hour.toString();
      _minute = selectedTime.minute.toString();
      _time = _hour + ' : ' + _minute;
      _timeController.text = _time;
      _timeController.text = formatDate(
          DateTime(2019, 08, 1, selectedTime.hour, selectedTime.minute),
          [hh, ':', nn, " ", am]).toString();
    });}

```

初始化 TimeOfDay 选择器类，将我们选择的时间保存在 _selecteTime 函数中。

{% asset_img 6.png 示意图 width="400" %}

> 在点击 TextFormField 时，我们调用一个 _selectTime 函数，它将显示日期和时间选择器以保存选择的时间。

```
InkWell(
  onTap: () {
    _selectTime(context);
  },
  child: Container(
    margin: EdgeInsets.only(top: 30),
    width: _width / 1.7,
    height: _height / 9,
    alignment: Alignment.center,
    decoration: BoxDecoration(color: Colors.grey[200]),
    child: TextFormField(
      style: TextStyle(fontSize: 40),
      textAlign: TextAlign.center,
      onSaved: (String val) {
        _setTime = val;
      },
      enabled: false,
      keyboardType: TextInputType.text,
      controller: _timeController,
      decoration: InputDecoration(
          disabledBorder:
              UnderlineInputBorder(borderSide: BorderSide.none),
          // labelText: 'Time',
          contentPadding: EdgeInsets.all(5)),
    ),
  ),
),
```

### 代码实现：

```
import 'package:date_format/date_format.dart';
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

class DateTimePicker extends StatefulWidget {
  @override
  _DateTimePickerState createState() => _DateTimePickerState();
}

class _DateTimePickerState extends State<DateTimePicker> {
  double _height;
  double _width;

  String _setTime, _setDate;

  String _hour, _minute, _time;

  String dateTime;

  DateTime selectedDate = DateTime.now();

  TimeOfDay selectedTime = TimeOfDay(hour: 00, minute: 00);

  TextEditingController _dateController = TextEditingController();
  TextEditingController _timeController = TextEditingController();

  Future<Null> _selectDate(BuildContext context) async {
    final DateTime picked = await showDatePicker(
        context: context,
        initialDate: selectedDate,
        initialDatePickerMode: DatePickerMode.day,
        firstDate: DateTime(2015),
        lastDate: DateTime(2101));
    if (picked != null)
      setState(() {
        selectedDate = picked;
        _dateController.text = DateFormat.yMd().format(selectedDate);
      });
  }

  Future<Null> _selectTime(BuildContext context) async {
    final TimeOfDay picked = await showTimePicker(
      context: context,
      initialTime: selectedTime,
    );
    if (picked != null)
      setState(() {
        selectedTime = picked;
        _hour = selectedTime.hour.toString();
        _minute = selectedTime.minute.toString();
        _time = _hour + ' : ' + _minute;
        _timeController.text = _time;
        _timeController.text = formatDate(
            DateTime(2019, 08, 1, selectedTime.hour, selectedTime.minute),
            [hh, ':', nn, " ", am]).toString();
      });
  }

  @override
  void initState() {
    _dateController.text = DateFormat.yMd().format(DateTime.now());

    _timeController.text = formatDate(
        DateTime(2019, 08, 1, DateTime.now().hour, DateTime.now().minute),
        [hh, ':', nn, " ", am]).toString();
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    _height = MediaQuery.of(context).size.height;
    _width = MediaQuery.of(context).size.width;
    dateTime = DateFormat.yMd().format(DateTime.now());
    return Scaffold(
      appBar: AppBar(
        centerTitle: true,
        title: Text('Date time picker'),
      ),
      body: Container(
        width: _width,
        height: _height,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.center,
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: <Widget>[
            Column(
              children: <Widget>[
                Text(
                  'Choose Date',
                  style: TextStyle(
                      fontStyle: FontStyle.italic,
                      fontWeight: FontWeight.w600,
                      letterSpacing: 0.5),
                ),
                InkWell(
                  onTap: () {
                    _selectDate(context);
                  },
                  child: Container(
                    width: _width / 1.7,
                    height: _height / 9,
                    margin: EdgeInsets.only(top: 30),
                    alignment: Alignment.center,
                    decoration: BoxDecoration(color: Colors.grey[200]),
                    child: TextFormField(
                      style: TextStyle(fontSize: 40),
                      textAlign: TextAlign.center,
                      enabled: false,
                      keyboardType: TextInputType.text,
                      controller: _dateController,
                      onSaved: (String val) {
                        _setDate = val;
                      },
                      decoration: InputDecoration(
                          disabledBorder:
                              UnderlineInputBorder(borderSide: BorderSide.none),
                          // labelText: 'Time',
                          contentPadding: EdgeInsets.only(top: 0.0)),
                    ),
                  ),
                ),
              ],
            ),
            Column(
              children: <Widget>[
                Text(
                  'Choose Time',
                  style: TextStyle(
                      fontStyle: FontStyle.italic,
                      fontWeight: FontWeight.w600,
                      letterSpacing: 0.5),
                ),
                InkWell(
                  onTap: () {
                    _selectTime(context);
                  },
                  child: Container(
                    margin: EdgeInsets.only(top: 30),
                    width: _width / 1.7,
                    height: _height / 9,
                    alignment: Alignment.center,
                    decoration: BoxDecoration(color: Colors.grey[200]),
                    child: TextFormField(
                      style: TextStyle(fontSize: 40),
                      textAlign: TextAlign.center,
                      onSaved: (String val) {
                        _setTime = val;
                      },
                      enabled: false,
                      keyboardType: TextInputType.text,
                      controller: _timeController,
                      decoration: InputDecoration(
                          disabledBorder:
                              UnderlineInputBorder(borderSide: BorderSide.none),
                          // labelText: 'Time',
                          contentPadding: EdgeInsets.all(5)),
                    ),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

结论 ：

在这篇文章中，我讲解了一个日期时间选择器的demo，大家可以根据自己的情况进行修改和实验，这个小介绍来自我们这边的日期时间选择器。
我希望这篇博客可以为你在 Flutter 项目中尝试使用日期时间选择器提供足够的信息。 所以请尝试一下。