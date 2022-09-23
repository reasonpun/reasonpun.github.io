---
title: 用于Flutter的令人惊叹的票务小工具
date: 2022-09-23 16:16:19
categories:
- Flutter
- Widget
tags:
- Flutter
- Widget
---

{% asset_img 1.png 示意图 width="400" %}

在您的flutter应用程序中添加漂亮的Ticket UI。

<!--more-->

在这篇文章中，我们将创建一个看起来像票据的小部件。我们可以在我们的应用程序中的任何地方使用该部件，使我们的应用程序看起来很时尚。我们将使用CustomPainter来绘制票务UI小部件。

⚙️让我们创建一个基础小部件
我们将首先创建一个TicketUi小部件，我们将把它变成一个像票据一样的小部件。

```dart
import 'package:flutter/material.dart';

class TicketUi extends StatelessWidget {
  const TicketUi({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Container(
            height: 220,
            margin: const EdgeInsets.all(16),
            width: MediaQuery.of(context).size.width,
            child: Container(),
          ),
        ),
      ),
    );
  }
}
```

稍后，我们将为上述小组件添加一个CustomPainter。CustomPainter将把这个小部件变成一个漂亮的票据用户界面。

🎨 实现自定义绘画器
让我们创建一个CustomPainter根类，并将其命名为TicketPainter。我们将在TicketPainter类的构造函数中传递背景颜色和边界颜色。

我们还为类定义了一些变量。这将被用来绘制门票的用户界面。

```dart
import 'package:flutter/material.dart';

class TicketPainter extends CustomPainter {
  final Color borderColor;
  final Color bgColor;
  
  TicketPainter({
    required this.bgColor,
    required this.borderColor,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final maxWidth = size.width;
    final maxHeight = size.height;

    final paintBg = Paint()
      ..style = PaintingStyle.fill
      ..strokeCap = StrokeCap.round
      ..color = bgColor;

    final paintBorder = Paint()
      ..strokeWidth = 1
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round
      ..color = borderColor;

    final paintDottedLine = Paint()
      ..color = borderColor.withOpacity(0.5)
      ..strokeWidth = 1.2;

    var path = Path();
    
  }

  // Since this Sky painter has no fields, it always paints
  // the same thing and semantics information is the same.
  // Therefore we return false here. If we had fields (set
  // from the constructor) then we would return true if any
  // of them differed from the same fields on the oldDelegate.
  @override
  bool shouldRepaint(TicketPainter oldDelegate) => false;

  @override
  bool shouldRebuildSemantics(TicketPainter oldDelegate) => false;
}
```

#### 绘制门票小部件 ✍️

现在我们将定义门票切口和半径的大小。你可以根据你的需要来改变这个。我们还将定义虚线的大小。

```dart
class TicketPainter extends CustomPainter {
  final Color borderColor;
  final Color bgColor;
  
  static const _cornerGap = 20.0;
  static const _cutoutRadius = 20.0;
  static const _cutoutDiameter = _cutoutRadius * 2;

  TicketPainter({
    required this.bgColor,
    required this.borderColor,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final maxWidth = size.width;
    final maxHeight = size.height;
    
    final cutoutStartPos = maxHeight - maxHeight * 0.2;
    final leftCutoutStartY = cutoutStartPos;
    final rightCutoutStartY = cutoutStartPos - _cutoutDiameter;
    final dottedLineY = cutoutStartPos - _cutoutRadius;
    double dottedLineStartX = _cutoutRadius;
    final double dottedLineEndX = maxWidth - _cutoutRadius;
    const double dashWidth = 8.5;
    const double dashSpace = 4;

    final paintBg = Paint()
      ..style = PaintingStyle.fill
      ..strokeCap = StrokeCap.round
      ..color = bgColor;

    final paintBorder = Paint()
      ..strokeWidth = 1
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round
      ..color = borderColor;

    final paintDottedLine = Paint()
      ..color = borderColor.withOpacity(0.5)
      ..strokeWidth = 1.2;

    var path = Path();
  }

  @override
  bool shouldRepaint(TicketPainter oldDelegate) => false;

  @override
  bool shouldRebuildSemantics(TicketPainter oldDelegate) => false;
}
```

上面我们已经定义了不同UI组件的X、Y坐标。这个X、Y坐标将被用来在你的小部件中绘制切口和虚线的路径。

现在我们将实现绘制切口和角弧的方法。🖌

```dart
class TicketPainter extends CustomPainter {
  final Color borderColor;
  final Color bgColor;
  
  static const _cornerGap = 20.0;
  static const _cutoutRadius = 20.0;
  static const _cutoutDiameter = _cutoutRadius * 2;

  TicketPainter({
    required this.bgColor,
    required this.borderColor,
  });

  @override
  void paint(Canvas canvas, Size size) {
    ///Removed code for redability
  }

  _drawCutout(Path path, double startX, double endY) {
    path.arcToPoint(
      Offset(startX, endY),
      radius: const Radius.circular(_cutoutRadius),
      clockwise: false,
    );
  }

  _drawCornerArc(Path path, double endPointX, double endPointY) {
    path.arcToPoint(
      Offset(endPointX, endPointY),
      radius: const Radius.circular(_cornerGap),
    );
  }
  
  @override
  bool shouldRepaint(TicketPainter oldDelegate) => false;

  @override
  bool shouldRebuildSemantics(TicketPainter oldDelegate) => false;
}
```

现在，最后一步是添加路径，绘制出票据用户界面。🛣

{% asset_img 2.jpeg 示意图 width="400" %}

```dart
import 'package:flutter/material.dart';

class TicketPainter extends CustomPainter {
  final Color borderColor;
  final Color bgColor;

  static const _cornerGap = 20.0;
  static const _cutoutRadius = 20.0;
  static const _cutoutDiameter = _cutoutRadius * 2;

  TicketPainter({required this.bgColor, required this.borderColor});

  @override
  void paint(Canvas canvas, Size size) {
    final maxWidth = size.width;
    final maxHeight = size.height;

    final cutoutStartPos = maxHeight - maxHeight * 0.2;
    final leftCutoutStartY = cutoutStartPos;
    final rightCutoutStartY = cutoutStartPos - _cutoutDiameter;
    final dottedLineY = cutoutStartPos - _cutoutRadius;
    double dottedLineStartX = _cutoutRadius;
    final double dottedLineEndX = maxWidth - _cutoutRadius;
    const double dashWidth = 8.5;
    const double dashSpace = 4;

    final paintBg = Paint()
      ..style = PaintingStyle.fill
      ..strokeCap = StrokeCap.round
      ..color = bgColor;

    final paintBorder = Paint()
      ..strokeWidth = 1
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round
      ..color = borderColor;

    final paintDottedLine = Paint()
      ..color = borderColor.withOpacity(0.5)
      ..strokeWidth = 1.2;

    var path = Path();

    path.moveTo(_cornerGap, 0);
    path.lineTo(maxWidth - _cornerGap, 0);
    _drawCornerArc(path, maxWidth, _cornerGap);
    path.lineTo(maxWidth, rightCutoutStartY);
    _drawCutout(path, maxWidth, rightCutoutStartY + _cutoutDiameter);
    path.lineTo(maxWidth, maxHeight - _cornerGap);
    _drawCornerArc(path, maxWidth - _cornerGap, maxHeight);
    path.lineTo(_cornerGap, maxHeight);
    _drawCornerArc(path, 0, maxHeight - _cornerGap);
    path.lineTo(0, leftCutoutStartY);
    _drawCutout(path, 0.0, leftCutoutStartY - _cutoutDiameter);
    path.lineTo(0, _cornerGap);
    _drawCornerArc(path, _cornerGap, 0);

    canvas.drawPath(path, paintBg);
    canvas.drawPath(path, paintBorder);

    while (dottedLineStartX < dottedLineEndX) {
      canvas.drawLine(
        Offset(dottedLineStartX, dottedLineY),
        Offset(dottedLineStartX + dashWidth, dottedLineY),
        paintDottedLine,
      );
      dottedLineStartX += dashWidth + dashSpace;
    }
  }

  _drawCutout(Path path, double startX, double endY) {
    path.arcToPoint(
      Offset(startX, endY),
      radius: const Radius.circular(_cutoutRadius),
      clockwise: false,
    );
  }

  _drawCornerArc(Path path, double endPointX, double endPointY) {
    path.arcToPoint(
      Offset(endPointX, endPointY),
      radius: const Radius.circular(_cornerGap),
    );
  }

  @override
  bool shouldRepaint(TicketPainter oldDelegate) => false;

  @override
  bool shouldRebuildSemantics(TicketPainter oldDelegate) => false;
}
```

最后，在小工具上添加自定义绘画器
最后一步是把CustomPainter添加到我们的initialTicketUi小部件中。

```dart
import 'package:blogs/ticket_ui/ticket_painter.dart';
import 'package:flutter/material.dart';

class TicketUi extends StatelessWidget {
  const TicketUi({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: Container(
            height: 220,
            margin: const EdgeInsets.all(16),
            width: MediaQuery.of(context).size.width,
            child: CustomPaint(
              painter: TicketPainter(
                borderColor: Colors.black,
                bgColor: const Color(0xFFfed966),
              ),
              child: Container(),
            ),
          ),
        ),
      ),
    );
  }
}
```

🤩 我们已经创建了一个很棒的票据用户界面。

{% asset_img 3.png 示意图 width="400" %}

最后的票据用户界面与一些自定义的用户界面。你可以根据你的需要创建你自己的用户界面🥳。

{% asset_img 4.png 示意图 width="400" %}

```dart
import 'package:blogs/ticket_ui/horizontal_dotted_line.dart';
import 'package:blogs/ticket_ui/ticket_painter.dart';
import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class TicketUiScreen extends StatelessWidget {
  const TicketUiScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: ListView.builder(
          itemBuilder: (_, __) {
            return Container(
              height: 220,
              margin: const EdgeInsets.all(16),
              width: MediaQuery.of(context).size.width,
              child: CustomPaint(
                painter: TicketPainter(
                  borderColor: Colors.black,
                  bgColor: const Color(0xFFfed966),
                ),
                child: Container(
                  padding: const EdgeInsets.all(16),
                  child: Column(
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            'DEA-HYD',
                            style: GoogleFonts.poppins(
                              fontSize: 14,
                              fontWeight: FontWeight.w700,
                            ),
                          ),
                          Text(
                            'BH07',
                            style: GoogleFonts.poppins(
                              fontSize: 14,
                              fontWeight: FontWeight.w400,
                            ),
                          ),
                          Text(
                            '\$140',
                            style: GoogleFonts.poppins(
                              fontSize: 20,
                              fontWeight: FontWeight.w800,
                            ),
                          ),
                        ],
                      ),
                      const SizedBox(height: 16),
                      Row(
                        crossAxisAlignment: CrossAxisAlignment.center,
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            'May 30, 2022',
                            style: GoogleFonts.poppins(
                              fontSize: 12,
                              fontWeight: FontWeight.w400,
                            ),
                          ),
                          const Padding(
                            padding: EdgeInsets.fromLTRB(8, 4, 0, 4),
                            child: Icon(
                              Icons.circle_outlined,
                              size: 18,
                            ),
                          ),
                          Expanded(
                            child: Stack(
                              children: [
                                Positioned.fill(
                                  child: Align(
                                    alignment: Alignment.center,
                                    child: CustomPaint(
                                      painter: HorizontalDottedLinePainter(),
                                      size: const Size(double.infinity, 1),
                                    ),
                                  ),
                                ),
                                const Center(
                                  child: RotatedBox(
                                    quarterTurns: 1,
                                    child: Icon(
                                      Icons.airplanemode_on_rounded,
                                      color: Colors.black,
                                      size: 28,
                                    ),
                                  ),
                                ),
                              ],
                            ),
                          ),
                          const Padding(
                            padding: EdgeInsets.fromLTRB(0, 4, 8, 4),
                            child: Icon(
                              Icons.circle_outlined,
                              size: 18,
                            ),
                          ),
                          Text(
                            'May 30, 2022',
                            style: GoogleFonts.poppins(
                              fontSize: 12,
                              fontWeight: FontWeight.w400,
                            ),
                          ),
                        ],
                      ),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            '10:40AM',
                            style: GoogleFonts.poppins(
                              fontSize: 18,
                              fontWeight: FontWeight.w600,
                            ),
                          ),
                          Text(
                            '1h 30m',
                            style: GoogleFonts.poppins(
                              fontSize: 14,
                              fontWeight: FontWeight.w400,
                            ),
                          ),
                          Text(
                            '12:50AM',
                            style: GoogleFonts.poppins(
                              fontSize: 18,
                              fontWeight: FontWeight.w600,
                            ),
                          ),
                        ],
                      ),
                      const Spacer(),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Text(
                            'Indigo',
                            style: GoogleFonts.poppins(
                              fontSize: 16,
                              fontWeight: FontWeight.w600,
                            ),
                          ),
                          Container(
                            decoration: BoxDecoration(
                              borderRadius: BorderRadius.circular(10),
                              color: Colors.black.withOpacity(0.2),
                              border: Border.all(
                                color: Colors.black.withOpacity(0.5),
                              ),
                            ),
                            child: Padding(
                              padding: const EdgeInsets.symmetric(
                                horizontal: 8,
                                vertical: 2,
                              ),
                              child: Text(
                                'Cheapest',
                                style: GoogleFonts.poppins(
                                  fontSize: 14,
                                  fontWeight: FontWeight.w400,
                                ),
                              ),
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ),
            );
          },
          itemCount: 6,
        ),
      ),
    );
  }
}
```

{% asset_img 5.gif 示意图 width="400" %}

谢谢你，我希望这篇文章在某种程度上帮助了你，祝你愉快！👋

<!-- https://arkapp.medium.com/stunning-ticket-widget-for-flutter-f9851e135eb9 -->