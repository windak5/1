import 'package:flutter/material.dart';
import 'chat_room_screen.dart';


class ChatsScreen extends StatefulWidget {
  const ChatsScreen({super.key});

  @override
  State<ChatsScreen> createState() => _ChatsScreenState();
}

class _ChatsScreenState extends State<ChatsScreen> {
  final TransformationController _transformationController = TransformationController();

    // Статические списки сохраняют координаты в памяти при переключении вкладок
  static final List<Map<String, dynamic>> _spaceChats = [
    {'name': 'Павел Дуров', 'lastMessage': 'Классный мессенджер получается!', 'x': 250.0, 'y': 300.0},
    {'name': 'Matrix Новости', 'lastMessage': 'Протокол обновлен', 'x': 700.0, 'y': 250.0},
    {'name': 'Разработка чата', 'lastMessage': 'Flutter работает идеально', 'x': 450.0, 'y': 550.0},
  ];

  static final List<Map<String, dynamic>> _zones = [
    {
      'id': 1,
      'title': 'Работа',
      'x': 400.0,
      'y': 200.0,
      'w': 500.0,
      'h': 500.0,
      'color': const Color(0xFF2160FF),
      'isEditing': false,
    },
  ];


    String _getInitials(String name) {
    if (name.isEmpty) return '';
    final words = name.trim().split(' ');
    if (words.length > 1) {
      // ИСПРАВЛЕНО: Берем первую букву имени [0] и первую букву фамилии [0]
      return (words.first[0] + words.last[0]).toUpperCase();
    }
    return words.first[0].toUpperCase();
  }


  void _createNewZone(Offset globalPosition) {
    setState(() {
      _zones.add({
        'id': DateTime.now().millisecondsSinceEpoch,
        'title': 'Новая область ${_zones.length + 1}',
        'x': globalPosition.dx - 150,
        'y': globalPosition.dy - 150,
        'w': 300.0,
        'h': 300.0,
        'color': const Color(0xFF2160FF),
        'isEditing': false,
      });
    });
  }
  void _showZoneMenu(Map<String, dynamic> zone) {
    final TextEditingController titleController = TextEditingController(text: zone['title']);

    showDialog(
      context: context,
      builder: (context) {
        return Center(
          child: Container(
            width: 360,
            padding: const EdgeInsets.all(32.0),
            decoration: BoxDecoration(
              color: const Color(0xFF24303F),
              borderRadius: BorderRadius.circular(24),
              boxShadow: [
                BoxShadow(color: Colors.black.withOpacity(0.5), blurRadius: 25, offset: const Offset(0, 10)),
              ],
            ),
            child: Material(
              color: Colors.transparent,
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Text(
                    'Управление областью',
                    style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold, color: Colors.white),
                  ),
                  const SizedBox(height: 24),
                  TextField(
                    controller: titleController,
                    decoration: InputDecoration(
                      hintText: 'Название области',
                      hintStyle: const TextStyle(color: Color(0xFF7F8C8D)),
                      filled: true,
                      fillColor: const Color(0xFF17212B),
                      border: OutlineInputBorder(borderRadius: BorderRadius.circular(12), borderSide: BorderSide.none),
                      contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
                    ),
                    style: const TextStyle(color: Colors.white),
                  ),
                  const SizedBox(height: 20),
                  _buildMenuButton(
                    text: 'Сохранить название',
                    color: const Color(0xFF5288C1),
                    onTap: () {
                      setState(() {
                        zone['title'] = titleController.text.trim();
                      });
                      Navigator.pop(context);
                    }
                  ),
                  const SizedBox(height: 12),
                  _buildMenuButton(
                    text: 'Редактировать положение',
                    color: const Color(0xFF17212B),
                    onTap: () {
                      setState(() {
                        zone['isEditing'] = true;
                      });
                      Navigator.pop(context);
                    }
                  ),
                  const SizedBox(height: 12),
                  _buildMenuButton(
                    text: 'Изменить цвет (Круг)',
                    color: const Color(0xFF17212B),
                    onTap: () {
                      Navigator.pop(context);
                      _showColorWheelDialog(zone);
                    }
                  ),
                  const SizedBox(height: 16),
                  _buildMenuButton(
                    text: 'Удалить область',
                    color: Colors.red.withOpacity(0.8),
                    onTap: () {
                      setState(() {
                        _zones.removeWhere((z) => z['id'] == zone['id']);
                      });
                      Navigator.pop(context);
                    }
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }

  void _showColorWheelDialog(Map<String, dynamic> zone) {
    final List<Color> wheelColors = [
      const Color(0xFF2160FF), const Color(0xFFB833E1), const Color(0xFFFF2D55),
      const Color(0xFFFF9500), const Color(0xFFFFCC00), const Color(0xFF4CD964),
      const Color(0xFF5AC8FA),
    ];

    showDialog(
      context: context,
      builder: (context) {
        return Center(
          child: Container(
            width: 320,
            padding: const EdgeInsets.all(28.0),
            decoration: BoxDecoration(
              color: const Color(0xFF24303F),
              borderRadius: BorderRadius.circular(24),
            ),
            child: Material(
              color: Colors.transparent,
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  const Text(
                    'Выберите цвет',
                    style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold, color: Colors.white),
                  ),
                  const SizedBox(height: 24),
                  Stack(
                    alignment: Alignment.center,
                    children: [
                      Container(
                        width: 180,
                        height: 180,
                        decoration: const BoxDecoration(
                          shape: BoxShape.circle,
                          gradient: SweepGradient(
                            colors: [
                              Color(0xFF2160FF), Color(0xFFB833E1), Color(0xFFFF2D55),
                              Color(0xFFFF9500), Color(0xFFFFCC00), Color(0xFF4CD964),
                              Color(0xFF5AC8FA), Color(0xFF2160FF),
                            ],
                          ),
                        ),
                      ),
                      SizedBox(
                        width: 180,
                        height: 180,
                        child: Wrap(
                          alignment: WrapAlignment.center,
                          runAlignment: WrapAlignment.center,
                          spacing: 14,
                          runSpacing: 14,
                          children: wheelColors.map((color) {
                            return GestureDetector(
                              onTap: () {
                                setState(() {
                                  zone['color'] = color;
                                });
                                Navigator.pop(context);
                              },
                              child: Container(
                                width: 36,
                                height: 36,
                                decoration: BoxDecoration(
                                  color: color,
                                  shape: BoxShape.circle,
                                  border: Border.all(color: Colors.white, width: 2.5),
                                ),
                              ),
                            );
                          }).toList(),
                        ),
                      ),
                    ],
                  ),
                  const SizedBox(height: 24),
                  _buildMenuButton(
                    text: 'Отмена',
                    color: const Color(0xFF17212B),
                    onTap: () => Navigator.pop(context),
                  ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }

  Widget _buildMenuButton({required String text, required Color color, required VoidCallback onTap}) {
    return SizedBox(
      width: double.infinity,
      height: 44,
      child: ElevatedButton(
        onPressed: onTap,
        style: ElevatedButton.styleFrom(
          backgroundColor: color,
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
          elevation: 0,
        ),
        child: Text(text, style: const TextStyle(fontSize: 14, fontWeight: FontWeight.bold, color: Colors.white)),
      ),
    );
  }
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF111823),
      body: Stack(
        children: [
                    // Попробуйте этот вариант вместо предыдущего контейнера сетки
          Positioned.fill(
            child: SizedBox(
              width: 4000,
              height: 4000,
              child: CustomPaint(
                painter: DotGridPainter(
                  // Сделаем цвет временно ярче (0.4 вместо 0.15), чтобы точно её увидеть
                  dotColor: const Color(0xFF2F3E4E).withOpacity(0.4),
                  spacing: 40.0,
                ),
              ),
            ),
          ),


          GestureDetector(
            onDoubleTapDown: (details) => _createNewZone(details.localPosition),
            child: InteractiveViewer(
              alignPanAxis: false,
              clipBehavior: Clip.none,
              boundaryMargin: const EdgeInsets.all(5000),
              minScale: 0.1,
              maxScale: 2.5,
              child: SizedBox(
                width: 15000,
                height: 15000,
                child: Stack(
                  children: [
                    Positioned.fill(
                      child: CustomPaint(
                        painter: DotGridPainter(
                          // Спокойный, едва заметный цвет для точек
                          dotColor: const Color(0xFF2F3E4E).withOpacity(0.25),
                          spacing: 40.0,
                        ),
                      ),
                    ),
                    ..._zones.map((zone) {
                      final bool isEditing = zone['isEditing'] == true;
                      return Positioned(
                        left: zone['x'],
                        top: zone['y'],
                        child: GestureDetector(
                          onPanUpdate: isEditing ? null : (details) {
                            setState(() {
                              double newX = (zone['x'] as num).toDouble() + details.delta.dx;
                              double newY = (zone['y'] as num).toDouble() + details.delta.dy;
                              zone['x'] = newX.clamp(50.0, 14000.0);
                              zone['y'] = newY.clamp(50.0, 14000.0);
                            });
                          },
                          child: _buildZoneContainer(zone),
                        ),
                      );
                    }),
                    ..._spaceChats.map((chat) {
                      return Positioned(
                        left: chat['x'] as double,
                        top: chat['y'] as double,
                        child: GestureDetector(
                          onPanUpdate: (details) {
                            setState(() {
                              double newX = (chat['x'] as num).toDouble() + details.delta.dx;
                              double newY = (chat['y'] as num).toDouble() + details.delta.dy;
                              chat['x'] = newX.clamp(50.0, 14500.0);
                              chat['y'] = newY.clamp(50.0, 14500.0);
                            });
                          },
                          onTap: () {
                            Navigator.push(
                              context,
                              MaterialPageRoute(
                                builder: (context) => ChatRoomScreen(chatName: chat['name'] as String),
                              ),
                            );
                          },
                          child: _buildChatCard(chat),
                        ),
                      );
                    }),
                  ],
                ),
              ),
            ),
          ),
          const Positioned(
            top: 28,
            left: 28,
            child: Row(
              children: [
                Icon(Icons.blur_on_rounded, color: Color(0xFF2160FF), size: 26),
                SizedBox(width: 12),
                Text(
                  'WinChat',
                  style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 20, letterSpacing: 0.5),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }

  Widget _buildChatCard(Map<String, dynamic> chat) {
    return Material(
      color: Colors.transparent,
      child: Container(
        width: 220,
        padding: const EdgeInsets.all(16),
        decoration: BoxDecoration(
          color: const Color(0xFF1E293B).withOpacity(0.95),
          borderRadius: BorderRadius.circular(20),
          border: Border.all(color: const Color(0xFF2F3E4E).withOpacity(0.6), width: 1.5),
          boxShadow: [
            BoxShadow(color: Colors.black.withOpacity(0.3), blurRadius: 15, offset: const Offset(0, 8)),
          ],
        ),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Container(
              width: 52,
              height: 52,
              decoration: const BoxDecoration(
                shape: BoxShape.circle,
                gradient: LinearGradient(
                  colors: [Color(0xFFB833E1), Color(0xFF2160FF), Color(0xFF00CDAC)],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
              ),
              child: Center(
                child: Text(
                  _getInitials(chat['name'] as String),
                  style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 16),
                ),
              ),
            ),
            const SizedBox(height: 12),
            Text(chat['name'] as String, style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 15)),
            const SizedBox(height: 4),
            Text(chat['lastMessage'] as String, style: const TextStyle(color: Color(0xFF708499), fontSize: 12), maxLines: 1, overflow: TextOverflow.ellipsis),
          ],
        ),
      ),
    );
  }

      Widget _buildZoneContainer(Map<String, dynamic> zone) {
    final bool isEditing = zone['isEditing'] == true;
    final Color zoneColor = zone['color'] as Color;

    final bool hasChatsInside = _spaceChats.any((chat) {
      final double chatX = (chat['x'] as num).toDouble();
      final double chatY = (chat['y'] as num).toDouble();
      final double zoneX = (zone['x'] as num).toDouble();
      final double zoneY = (zone['y'] as num).toDouble();
      final double zoneW = (zone['w'] as num).toDouble();
      final double zoneH = (zone['h'] as num).toDouble();

      return chatX >= zoneX &&
             chatX <= (zoneX + zoneW) &&
             chatY >= zoneY &&
             chatY <= (zoneY + zoneH);
    });

    return Material(
      color: Colors.transparent,
      child: GestureDetector(
        onTap: () {
          if (isEditing) {
            setState(() {
              zone['isEditing'] = false;
            });
          }
        },
        child: Container(
          width: (zone['w'] as num).toDouble() + (isEditing ? 50 : 0),
          height: (zone['h'] as num).toDouble() + (isEditing ? 50 : 0),
          color: Colors.transparent,
          child: Stack(
            alignment: Alignment.center,
            children: [
              Container(
                width: (zone['w'] as num).toDouble(),
                height: (zone['h'] as num).toDouble(),
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  // Заливка берет выбранный цвет с минимальной прозрачностью (5%)
                  color: zoneColor.withOpacity(0.05),
                  borderRadius: BorderRadius.circular(28),
                  border: Border.all(
                    // Если чат внутри — увеличиваем видимость родного цвета до 55%
                    // Если пустая — оставляем базовые аккуратные 20%
                    color: (hasChatsInside || isEditing)
                        ? zoneColor.withOpacity(0.55)
                        : zoneColor.withOpacity(0.2),
                    width: isEditing ? 2.5 : 1.5,
                  ),
                  boxShadow: [
                    // Свечение тоже красится в выбранный цвет, но еле заметно (5% яркости)
                    if (hasChatsInside || isEditing)
                      BoxShadow(
                        color: zoneColor.withOpacity(0.05),
                        blurRadius: 10,
                        spreadRadius: 0,
                      ),
                  ],
                ),

                child: Stack(
                  children: [
                    Align(
                      alignment: Alignment.topCenter,
                      child: Container(
                        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                        decoration: BoxDecoration(
                          color: const Color(0xFF1F2430).withOpacity(0.6),
                          borderRadius: BorderRadius.circular(8),
                        ),
                        child: Text(
                          zone['title'],
                          style: TextStyle(
                            // Если чат внутри или идет редактирование — делаем родной цвет текста ярким (85% видимости)
                            // Если область пустая — оставляем его базовым и спокойным
                            color: (hasChatsInside || isEditing)
                                ? zoneColor.withOpacity(0.85)
                                : zoneColor.withOpacity(0.5),
                            fontWeight: FontWeight.bold,
                            fontSize: 16,
                          ),
                        ),

                      ),
                    ),
                    Positioned(
                      top: 0,
                      right: 0,
                      child: IconButton(
                        icon: const Icon(Icons.more_horiz, color: Colors.white, size: 24),
                        onPressed: () => _showZoneMenu(zone),
                      ),
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }



  Widget _buildArrowIndicator(IconData icon, double degrees) {
    return Container(
      padding: const EdgeInsets.all(5),
      decoration: const BoxDecoration(color: Color(0xFF24303F), shape: BoxShape.circle),
      child: RotationTransition(
        turns: AlwaysStoppedAnimation(degrees / 360),
        child: Icon(icon, color: Colors.white, size: 14),
      ),
    );
  }
}

class GridPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = const Color(0xFF1F2937).withOpacity(0.25)
      ..strokeWidth = 1.0;
    const double gridSpace = 60.0;
    for (double i = 0; i < size.width; i += gridSpace) {
      canvas.drawLine(Offset(i, 0), Offset(i, size.height), paint);
    }
    for (double i = 0; i < size.height; i += gridSpace) {
      canvas.drawLine(Offset(0, i), Offset(size.width, i), paint);
    }
  }
  @override
    bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
  }

class DotGridPainter extends CustomPainter {
  final Color dotColor;
  final double spacing;

  DotGridPainter({required this.dotColor, this.spacing = 40.0});

  @override
  void paint(Canvas canvas, Size size) {
    // Используем максимально легкий стиль только для отрисовки точек
    final paint = Paint()
      ..color = dotColor
      ..style = PaintingStyle.fill;

    for (double x = 0; x < size.width; x += spacing) {
      for (double y = 0; y < size.height; y += spacing) {
        // Рисуем аккуратную маленькую точку радиусом 1.2 пикселя
        canvas.drawCircle(Offset(x, y), 1.2, paint);
      }
    }
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}



