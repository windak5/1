import 'package:flutter/material.dart';

bool _isThoughtOpenOnPc = false;


class ThoughtsFeedScreen extends StatefulWidget {
  final Function(bool)? onThoughtToggle;

  const ThoughtsFeedScreen({super.key, this.onThoughtToggle});

  @override
  State<ThoughtsFeedScreen> createState() => _ThoughtsFeedScreenState();
}

class _ThoughtsFeedScreenState extends State<ThoughtsFeedScreen> {
  // Список анонимных мыслей (в будущем пойдет из Matrix/Бэкенда)
    // Обновленный список анонимных мыслей с заголовками и комментариями
  final List<Map<String, dynamic>> _thoughts = [
    {
      'title': 'Тишина в 22 года',
      'text': 'Иногда я просто сижу в машине по 20 минут перед тем как зайти домой, просто чтобы побыть в тишине. Окружающие думают, что у меня проблемы, но мне просто нужен барьер между работой и домом. Это нормально в 22 года или я уже ментально устал от жизни?',
      'likes': 142,
      'isLiked': false,
      'time': '5 мин назад',
      'color': const Color(0xFF1E2938),
      'commentsList': [
        'Это абсолютно нормально, бро. Делаю так же в свои 30.',
        'Машина — это как личная безопасная капсула, понимаю.',
      ],
    },
    {
      'title': 'Курсы за 50к',
      'text': 'Купил курс по программированию за 50к, надеялся на супер мотивацию и структурированный материал. В итоге весь день смотрю мемы в телеге, закрываю вкладки с лекциями, а тему Senior-разработки учу по бесплатным видосам индийцев на ютубе. Гений мысли и маркетинга.',
      'likes': 89,
      'isLiked': true,
      'time': '20 мин назад',
      'color': const Color(0xFF1A2332),
      'commentsList': [
        'Индийцы на ютубе — лучшие менторы в мире, смирись) 😂',
      ],
    },
  ];


  // Контроллер для ввода новой мысли
  final TextEditingController _textController = TextEditingController();
    // Хранит выбранную мысль, которая сейчас открыта на ПК
    Map<String, dynamic>? _selectedThought;


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
    // Увеличили порог до 900, чтобы браузер на ПК гарантированно открывал двухпанельный режим
    final bool isMobile = MediaQuery.of(context).size.width < 900;

    return Scaffold(
      backgroundColor: const Color(0xFF0E131A),
      body: SafeArea(
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ЛЕВАЯ ЧАСТЬ: Лента заголовков и карточек
            Expanded(
              flex: 4,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Шапка экрана Мысли
                  Padding(
                    padding: const EdgeInsets.only(top: 24, left: 24, bottom: 12, right: 16),
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Row(
                          children: [
                            const Icon(Icons.wb_twilight_rounded, color: Color(0xFF00F5D4), size: 24),
                            const SizedBox(width: 8),
                            const Text(
                              'Мысли',
                              style: TextStyle(color: Colors.white, fontSize: 22, fontWeight: FontWeight.bold),
                            ),
                          ],
                        ),
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
                  // Список карточек
                  Expanded(
                    child: ListView.builder(
                      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                      itemCount: _thoughts.length,
                      itemBuilder: (context, index) {
                        final item = _thoughts[index];
                        final bool isLiked = item['isLiked'] ?? false;

                        return GestureDetector(
                          onTap: () {
                            if (isMobile) {
                              // На мобильных устройствах открываем карточку на весь экран
                              Navigator.push(
                                context,
                                MaterialPageRoute(
                                  builder: (context) => Scaffold(
                                    backgroundColor: const Color(0xFF0E131A),
                                    body: SafeArea(
                                      child: _buildDetailedThoughtPanel(), // Открываем панель напрямую
                                    ),
                                  ),
                                ),
                              );
                            } else {
                              // На ПК открываем боковую панель справа
                              setState(() {
                                _selectedThought = item;
                                if (widget.onThoughtToggle != null) {
                                  widget.onThoughtToggle!(true);
                                }
                              });
                            }
                          },
                          child: Container(
                            margin: const EdgeInsets.only(bottom: 12),
                            padding: const EdgeInsets.all(16),
                            decoration: BoxDecoration(
                              color: item['color'],
                              borderRadius: BorderRadius.circular(20),
                              border: Border.all(
                                color: _selectedThought == item
                                    ? const Color(0xFF00F5D4).withOpacity(0.5)
                                    : const Color(0xFF2F3E4E).withOpacity(0.2),
                                width: 1,
                              ),
                            ),
                            child: Column(
                              crossAxisAlignment: CrossAxisAlignment.start,
                              children: [
                                Text(
                                  item['title'] ?? 'Без названия',
                                  style: const TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.bold),
                                ),
                                const SizedBox(height: 6),
                                Text(
                                  item['text'],
                                  maxLines: 2,
                                  overflow: TextOverflow.ellipsis,
                                  style: TextStyle(color: Colors.white.withOpacity(0.7), fontSize: 13, height: 1.4),
                                ),
                                const SizedBox(height: 14),
                                Row(
                                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                                  children: [
                                    Text(item['time'], style: TextStyle(color: Colors.white.withOpacity(0.25), fontSize: 11)),
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
                                          color: isLiked ? Colors.red.withOpacity(0.1) : Colors.transparent,
                                          borderRadius: BorderRadius.circular(8),
                                        ),
                                        child: Row(
                                          children: [
                                            Icon(
                                              isLiked ? Icons.favorite_rounded : Icons.favorite_border_rounded,
                                              color: isLiked ? const Color(0xFFFF4D4D) : Colors.white.withOpacity(0.4),
                                              size: 16,
                                            ),
                                            const SizedBox(width: 4),
                                            Text(
                                              '${item['likes']}',
                                              style: TextStyle(
                                                color: isLiked ? const Color(0xFFFF4D4D) : Colors.white.withOpacity(0.4),
                                                fontSize: 12,
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
                          ),
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),

            // ПРАВАЯ ЧАСТЬ: Полноценное окно детализации с комментариями (Только на ПК)
            if (!isMobile)
              Expanded(
                flex: 6,
                child: Container(
                  color: const Color(0xFF0E131A),
                  child: _buildDetailedThoughtPanel(),
                ),
              ),
          ],
        ),
      ),
    );
  }

    // Шаг 3. Правая панель: Детальный просмотр анонимной мысли и комментариев
  Widget _buildDetailedThoughtPanel() {
    if (_selectedThought == null) {
      return Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.wb_twilight_rounded, size: 64, color: const Color(0xFF00F5D4).withOpacity(0.15)),
            const SizedBox(height: 16),
            Text('Выберите мысль, чтобы прочитать полностью', style: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 15)),
          ],
        ),
      );
    }

    final List<String> comments = List<String>.from(_selectedThought!['commentsList'] ?? []);
    final TextEditingController commentController = TextEditingController();

    return Container(
      decoration: const BoxDecoration(border: Border(left: BorderSide(color: Color(0xFF1E2938), width: 1))),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Шапка панели
          Container(
            padding: const EdgeInsets.all(24),
            color: const Color(0xFF111823),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Expanded(child: Text(_selectedThought!['title'] ?? 'Без названия', style: const TextStyle(color: Colors.white, fontSize: 20, fontWeight: FontWeight.bold))),
                IconButton(icon: const Icon(Icons.close_rounded, color: Colors.grey), onPressed: () => setState(() => _selectedThought = null)),
              ],
            ),
          ),

          // Контент: мысль и комментарии
          Expanded(
            child: ListView(
              padding: const EdgeInsets.all(24),
              children: [
                Text(_selectedThought!['text'], style: const TextStyle(color: Colors.white, fontSize: 15, height: 1.5)),
                const SizedBox(height: 12),
                Text(_selectedThought!['time'], style: TextStyle(color: Colors.white.withOpacity(0.2), fontSize: 12)),
                const SizedBox(height: 32),
                Text('Комментарии (${comments.length})', style: const TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.bold)),
                const SizedBox(height: 16),
                if (comments.isEmpty)
                  Text('Здесь пока нет комментариев. Напишите первый!', style: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 14))
                else
                  ...comments.map((comment) => Container(
                        margin: const EdgeInsets.only(bottom: 10),
                        padding: const EdgeInsets.all(14),
                        decoration: BoxDecoration(color: const Color(0xFF1E2938).withOpacity(0.5), borderRadius: BorderRadius.circular(12)),
                        child: Text(comment, style: const TextStyle(color: Colors.white, fontSize: 14)),
                      )),
              ],
            ),
          ),

          // Поле ввода комментария
          Padding(
            padding: const EdgeInsets.all(24),
            child: TextField(
              controller: commentController,
              style: const TextStyle(color: Colors.white),
              decoration: InputDecoration(
                hintText: 'Оставить анонимный комментарий...',
                suffixIcon: IconButton(
                  icon: const Icon(Icons.send_rounded, color: Color(0xFF00F5D4)),
                  onPressed: () {
                    final text = commentController.text.trim();
                    if (text.isNotEmpty) {
                      setState(() {
                        _selectedThought!['commentsList'] = [...comments, text];
                      });
                      commentController.clear();
                    }
                  },
                ),
              ),
            ),
          ),
        ],
      ),
    );
  }

    // ШАГ 1: Исправленный мобильный экран (Без стрелочки, работает через крестик)
  Widget _buildMobileThoughtView(Map<String, dynamic> item) {
    _selectedThought = item; // Подменяем выбранную мысль

    // Возвращаем просто чистую панель без верхнего AppBar.
    // Теперь крестик внутри неё будет сам закрывать экран!
    return _buildDetailedThoughtPanel();
  }
}
