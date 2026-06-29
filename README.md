import 'package:flutter/material.dart';

class ThoughtsFeedScreen extends StatefulWidget {
  const ThoughtsFeedScreen({super.key});

  @override
  State<ThoughtsFeedScreen> createState() => _ThoughtsFeedScreenState();
}

class _ThoughtsFeedScreenState extends State<ThoughtsFeedScreen> {
  // Список анонимных мыслей (в будущем пойдет из Matrix/Бэкенда)
  final List<Map<String, dynamic>> _thoughts = [
    {
      'text': 'Иногда я просто сижу в машине по 20 минут перед тем как зайти домой, просто чтобы побыть в тишине. Это нормально в 22 года?',
      'likes': 142,
      'comments': 24,
      'isLiked': false,
      'time': '5 мин назад',
      'color': const Color(0xFF1E2938),
    },
    {
      'text': 'Купил курс по программированию за 50к, в итоге весь день смотрю мемы и учусь по бесплатным видосам на ютубе. Гений мысли.',
      'likes': 89,
      'comments': 12,
      'isLiked': true,
      'time': '20 мин назад',
      'color': const Color(0xFF1A2332),
    },
    {
      'text': 'Смешно, как мы все притворяемся взрослыми и серьезными на работе, хотя внутри каждый второй хочет просто построить шалаш из одеял и пить какао.',
      'likes': 315,
      'comments': 56,
      'isLiked': false,
      'time': '1 час назад',
      'color': const Color(0xFF243141),
    },
  ];

  // Контроллер для ввода новой мысли
  final TextEditingController _textController = TextEditingController();

  // Функция для публикации новой мысли
  void _publishThought() {
    if (_textController.text.trim().isEmpty) return;
    setState(() {
      _thoughts.insert(0, {
        'text': _textController.text,
        'likes': 0,
        'comments': 0,
        'isLiked': false,
        'time': 'Только что',
        'color': const Color(0xFF1E2938),
      });
      _textController.clear();
    });
    Navigator.pop(context); // Закрываем окошко ввода
  }
  // Открытие окна для написания новой анонимной мысли
  void _showAddThoughtSheet() {
    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      backgroundColor: const Color(0xFF1E2938),
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
      ),
      builder: (context) {
        return Padding(
          padding: EdgeInsets.only(
            bottom: MediaQuery.of(context).viewInsets.bottom,
            left: 20,
            right: 20,
            top: 24,
          ),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              const Text(
                'Поделиться мыслью анонимно',
                style: TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold),
              ),
              const SizedBox(height: 14),
              TextField(
                controller: _textController,
                maxLines: 4,
                maxLength: 200,
                style: const TextStyle(color: Colors.white),
                decoration: InputDecoration(
                  hintText: 'О чём вы думаете? Никто не узнает, что это вы...',
                  hintStyle: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 14),
                  filled: true,
                  fillColor: const Color(0xFF0E131A),
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(16),
                    borderSide: BorderSide.none,
                  ),
                  counterStyle: TextStyle(color: Colors.white.withOpacity(0.3)),
                ),
              ),
              const SizedBox(height: 10),
              Align(
                alignment: Alignment.centerRight,
                child: ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: const Color(0xFF00F5D4), // Наш фирменный мятный
                    foregroundColor: const Color(0xFF0E131A),
                    padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
                    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                    elevation: 5,
                    shadowColor: const Color(0xFF00F5D4).withOpacity(0.4),
                  ),
                  onPressed: _publishThought,
                  child: const Text('Опубликовать', style: TextStyle(fontWeight: FontWeight.bold)),
                ),
              ),
              const SizedBox(height: 20),
            ],
          ),
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF0E131A),
      body: Stack(
        children: [
          Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Шапка экрана Эфир
              Padding(
                padding: const EdgeInsets.only(top: 24, left: 88, bottom: 12, right: 16),
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Row(
                      children: [
                        const Icon(Icons.wb_twilight_rounded, color: Color(0xFF00F5D4), size: 24),
                        const SizedBox(width: 8),
                        Text(
                          'Мысли',
                          style: TextStyle(
                            color: Colors.white.withOpacity(0.95),
                            fontSize: 22,
                            fontWeight: FontWeight.bold,
                            letterSpacing: 0.5,
                          ),
                        ),
                      ],
                    ),
                    // Маленькая круглая кнопка "+" для добавления мысли
                    GestureDetector(
                      onTap: _showAddThoughtSheet,
                      child: Container(
                        padding: const EdgeInsets.all(8),
                        decoration: BoxDecoration(
                          shape: BoxShape.circle,
                          color: const Color(0xFF1E2938),
                          border: Border.all(color: const Color(0xFF00F5D4).withOpacity(0.3)),
                        ),
                        child: const Icon(Icons.add, color: Color(0xFF00F5D4), size: 20),
                      ),
                    ),
                  ],
                ),
              ),
              // Список анонимных мыслей
              Expanded(
                child: ListView.builder(
                  padding: const EdgeInsets.only(left: 16, right: 16, top: 8, bottom: 90),
                  itemCount: _thoughts.length,
                  itemBuilder: (context, index) {
                    final item = _thoughts[index];
                    final bool isLiked = item['isLiked'] ?? false;

                    return Container(
                      margin: const EdgeInsets.only(bottom: 12),
                      padding: const EdgeInsets.all(16),
                      decoration: BoxDecoration(
                        color: item['color'],
                        borderRadius: BorderRadius.circular(20),
                        border: Border.all(color: const Color(0xFF2F3E4E).withOpacity(0.2), width: 1),
                      ),
                      child: Column(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          // Текст мысли
                          Text(
                            item['text'],
                            style: const TextStyle(color: Colors.white, fontSize: 14, height: 1.4, fontWeight: FontWeight.w400),
                          ),
                          const SizedBox(height: 14),
                          // Нижняя панель карточки (время, лайки, комменты)
                          Row(
                            mainAxisAlignment: MainAxisAlignment.spaceBetween,
                            children: [
                              Text(
                                item['time'],
                                style: TextStyle(color: Colors.white.withOpacity(0.25), fontSize: 11),
                              ),
                              Row(
                                children: [
                                  // Кнопка комментариев
                                  Row(
                                    children: [
                                      Icon(Icons.chat_bubble_outline_rounded, color: Colors.white.withOpacity(0.4), size: 16),
                                      const SizedBox(width: 4),
                                      Text('${item['comments']}', style: TextStyle(color: Colors.white.withOpacity(0.4), fontSize: 12)),
                                    ],
                                  ),
                                  const SizedBox(width: 16),
                                  // Кнопка лайка (горит мятным, если лайкнуто)
                                  GestureDetector(
                                    onTap: () {
                                      setState(() {
                                        item['isLiked'] = !isLiked;
                                        item['likes'] += isLiked ? -1 : 1;
                                      });
                                    },
                                    child: AnimatedContainer(
                                      duration: const Duration(milliseconds: 200),
                                      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                                      decoration: BoxDecoration(
                                        color: isLiked ? const Color(0xFF00F5D4).withOpacity(0.1) : Colors.transparent,
                                        borderRadius: BorderRadius.circular(8),
                                      ),
                                      child: Row(
                                        children: [
                                          Icon(
                                            isLiked ? Icons.favorite_rounded : Icons.favorite_border_rounded,
                                            color: isLiked ? const Color(0xFF00F5D4) : Colors.white.withOpacity(0.4),
                                            size: 16,
                                          ),
                                          const SizedBox(width: 4),
                                          Text(
                                            '${item['likes']}',
                                            style: TextStyle(
                                              color: isLiked ? const Color(0xFF00F5D4) : Colors.white.withOpacity(0.4),
                                              fontSize: 12,
                                              fontWeight: isLiked ? FontWeight.bold : FontWeight.normal,
                                            ),
                                          ),
                                        ],
                                      ),
                                    ),
                                  ),
                                ],
                              ),
                            ],
                          ),
                        ],
                      ),
                    );
                  },
                ),
              ),
            ],
          ),
        ],
      ),
    );
  }
}
