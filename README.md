import 'package:flutter/material.dart';
import 'chat_room_screen.dart';

class ChatsScreen extends StatefulWidget {
  const ChatsScreen({super.key});

  @override
  State<ChatsScreen> createState() => _ChatsScreenState();
}

class _ChatsScreenState extends State<ChatsScreen> {
  final List<Map<String, dynamic>> _spaceChats = [
    {'name': 'Павел Дуров', 'lastMessage': 'Классный мессенджер получается!', 'x': 250.0, 'y': 300.0},
    {'name': 'Matrix Новости', 'lastMessage': 'Протокол обновлен', 'x': 700.0, 'y': 250.0},
    {'name': 'Разработка чата', 'lastMessage': 'Flutter работает идеально', 'x': 450.0, 'y': 550.0},
  ];

  final List<Map<String, dynamic>> _zones = [
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
          GestureDetector(
            onDoubleTapDown: (details) => _createNewZone(details.localPosition),
            child: InteractiveViewer(
              boundaryMargin: const EdgeInsets.all(1500),
              minScale: 0.3,
              maxScale: 2.0,
              child: SizedBox(
                width: 3000,
                height: 3000,
                child: Stack(
                  children: [
                    Positioned.fill(
                      child: CustomPaint(painter: GridPainter()),
                    ),
                    ..._zones.map((zone) {
                      final bool isEditing = zone['isEditing'] == true;
                      return Positioned(
                        left: zone['x'],
                        top: zone['y'],
                        child: GestureDetector(
                          onPanUpdate: isEditing ? null : (details) {
                            setState(() {
                              zone['x'] += details.delta.dx;
                              zone['y'] += details.delta.dy;
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
                              chat['x'] = (chat['x'] as double) + details.delta.dx;
                              chat['y'] = (chat['y'] as double) + details.delta.dy;
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
                  'Пространство Matrix',
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
          width: zone['w'] + (isEditing ? 50 : 0),
          height: zone['h'] + (isEditing ? 50 : 0),
          color: Colors.transparent,
          child: Stack(
            alignment: Alignment.center,
            children: [
              Container(
                width: zone['w'],
                height: zone['h'],
                padding: const EdgeInsets.all(12),
                decoration: BoxDecoration(
                  color: const Color(0xFF1E293B).withOpacity(0.15),
                  borderRadius: BorderRadius.circular(28),
                  border: Border.all(
                    color: isEditing ? zoneColor : zoneColor.withOpacity(0.4),
                    width: isEditing ? 2.5 : 2,
                  ),
                ),
                child: Stack(
                  children: [
                    Align(
                      alignment: Alignment.topCenter,
                      child: Container(
                        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                        decoration: BoxDecoration(
                          color: const Color(0xFF24303F).withOpacity(0.6),
                          borderRadius: BorderRadius.circular(8),
                        ),
                        child: Text(zone['title'], style: TextStyle(color: zoneColor, fontWeight: FontWeight.bold, fontSize: 16)),
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
              if (isEditing) ...[
                Positioned(
                  right: 0,
                  child: GestureDetector(
                    onPanUpdate: (details) {
                      setState(() { zone['w'] = (zone['w'] + details.delta.dx).clamp(200.0, 900.0); });
                    },
                    child: _buildArrowIndicator(Icons.code_rounded, 0),
                  ),
                ),
                Positioned(
                  left: 0,
                  child: GestureDetector(
                    onPanUpdate: (details) {
                      setState(() {
                        final double oldW = zone['w'];
                        zone['w'] = (zone['w'] - details.delta.dx).clamp(200.0, 900.0);
                        zone['x'] += (oldW - zone['w']);
                      });
                    },
                    child: _buildArrowIndicator(Icons.code_rounded, 0),
                  ),
                ),
                Positioned(
                  bottom: 0,
                  child: GestureDetector(
                    onPanUpdate: (details) {
                      setState(() { zone['h'] = (zone['h'] + details.delta.dy).clamp(200.0, 900.0); });
                    },
                    child: _buildArrowIndicator(Icons.code_rounded, 90),
                  ),
                ),
                Positioned(
                  top: 0,
                  child: GestureDetector(
                    onPanUpdate: (details) {
                      setState(() {
                        final double oldH = zone['h'];
                        zone['h'] = (zone['h'] - details.delta.dy).clamp(200.0, 900.0);
                        zone['y'] += (oldH - zone['h']);
                      });
                    },
                    child: _buildArrowIndicator(Icons.code_rounded, 90),
                  ),
                ),
              ],
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
