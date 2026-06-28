import 'package:flutter/material.dart';
import 'chat_room_screen.dart';

class ChatsScreen extends StatefulWidget {
  const ChatsScreen({super.key});

  @override
  State<ChatsScreen> createState() => _ChatsScreenState();
}

class _ChatsScreenState extends State<ChatsScreen> {
  // Список чатов мессенджера
  final List<Map<String, dynamic>> _spaceChats = [
    {'name': 'Павел Дуров', 'lastMessage': 'Классный мессенджер получается!', 'unread': 2, 'time': '12:30'},
    {'name': 'Matrix Новости', 'lastMessage': 'Протокол обновлен до версии 1.11', 'unread': 0, 'time': 'Вчера'},
    {'name': 'Разработка чата', 'lastMessage': 'Flutter работает идеально', 'unread': 5, 'time': 'Позавчера'},
  ];

  // Данные для геймификации: «Серии переписок» (Стрики с огоньками)
  final List<Map<String, dynamic>> _chatStreaks = [
    {'name': 'Павел Дуров', 'days': 15, 'active': true},
    {'name': 'Разработка чата', 'days': 4, 'active': true},
    {'name': 'Matrix Новости', 'days': 120, 'active': false},
  ];

  // Вспомогательный метод для автоматического получения букв аватарки
  String _getInitials(String name) {
    List<String> names = name.split(" ");
    String initials = "";
    if (names.isNotEmpty && names[0].isNotEmpty) {
      initials += names[0][0];
    }
    if (names.length > 1 && names[1].isNotEmpty) {
      initials += names[1][0];
    }
    return initials.toUpperCase();
  }

    @override
  Widget build(BuildContext context) {
    // Проверяем, мобилка это или ПК (если ширина меньше 600px — это мобильный)
    final bool isMobile = MediaQuery.of(context).size.width < 600;

    return Scaffold(
      backgroundColor: const Color(0xFF0E131A), // Глубокий темный фон мессенджера
      body: SafeArea(
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Шапка главной страницы WinChat
            Padding(
              padding: const EdgeInsets.only(top: 16, left: 24, bottom: 12, right: 16),
              child: Row(
                children: [
                  const Icon(Icons.blur_on_rounded, color: Color(0xFF00F5D4), size: 24),
                  const SizedBox(width: 8),
                  Text(
                    'WinChat',
                    style: TextStyle(
                      color: Colors.white.withOpacity(0.95),
                      fontSize: 22,
                      fontWeight: FontWeight.bold,
                      letterSpacing: 0.5,
                    ),
                  ),
                ],
              ),
            ),

            // Если это ТЕЛЕФОН — выводим серии сверху горизонтальной лентой кружочков
            if (isMobile) _buildMobileStreaks(),

            // Главный контент страницы
            Expanded(
              child: Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Левая панель: Список чатов (на мобилке на весь экран, на ПК — колонка)
                  Expanded(
                    flex: 3,
                    child: ListView.builder(
                      padding: const EdgeInsets.only(left: 16, right: 16, top: 4, bottom: 90), // Запас под остров навигации
                      itemCount: _spaceChats.length,
                      itemBuilder: (context, index) {
                        final chat = _spaceChats[index];
                        return _buildChatCard(chat);
                      },
                    ),
                  ),

                  // Если это ПК — выводим справа красивый темный блок активных серий
                  if (!isMobile)
                    Expanded(
                      flex: 2,
                      child: Padding(
                        padding: const EdgeInsets.only(right: 24, left: 8, top: 4),
                        child: _buildPcStreakPanel(),
                      ),
                    ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }



    // 1. Виджет серий для МОБИЛЬНОГО телефона (Горизонтальные кружочки сверху)
    // 1. Виджет серий для МОБИЛЬНОГО телефона (Горизонтальные кружочки сверху)
    // 1. Виджет серий для МОБИЛЬНОГО телефона (Горизонтальные кружочки сверху)
    // 1. Виджет серий для МОБИЛЬНОГО телефона (Адаптирован под квадраты и огненный градиент)
  Widget _buildMobileStreaks() {
    return Container(
      height: 75,
      margin: const EdgeInsets.only(bottom: 8),
      child: ListView.builder(
        scrollDirection: Axis.horizontal,
        padding: const EdgeInsets.symmetric(horizontal: 16),
        itemCount: _chatStreaks.length,
        itemBuilder: (context, index) {
          final streak = _chatStreaks[index];
          return Padding(
            padding: const EdgeInsets.only(right: 16),
            child: Column(
              children: [
                Stack(
                  alignment: Alignment.bottomRight,
                  children: [
                    Container(
                      width: 44,
                      height: 44,
                      decoration: BoxDecoration(
                        // ИЗМЕНЕНИЕ: Строгая квадратная форма вместо круга
                        borderRadius: BorderRadius.circular(10),
                        color: const Color(0xFF1E2938),
                        border: Border.all(
                          color: streak['active'] ? const Color(0xFFFF4500) : const Color(0xFF2F3E4E),
                          width: 1.5,
                        ),
                      ),
                      child: Center(
                        child: Text(
                          _getInitials(streak['name']),
                          style: const TextStyle(color: Colors.white, fontSize: 12, fontWeight: FontWeight.bold),
                        ),
                      ),
                    ),
                    // Огонёк со счетчиком дней
                    Container(
                      padding: const EdgeInsets.symmetric(horizontal: 4, vertical: 1),
                      decoration: BoxDecoration(
                        color: const Color(0xFF17212B),
                        borderRadius: BorderRadius.circular(4), // Квадратная плашка
                        border: Border.all(color: streak['active'] ? const Color(0xFFFF3333).withOpacity(0.5) : Colors.transparent),
                      ),
                      child: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          // ИЗМЕНЕНИЕ: Градиентный огненный эффект для иконки
                          ShaderMask(
                            shaderCallback: (bounds) => const LinearGradient(
                              colors: [Color(0xFFFF8C00), Color(0xFFFF0000)], // От оранжевого к красному
                              begin: Alignment.topCenter,
                              end: Alignment.bottomCenter,
                            ).createShader(bounds),
                            child: const Icon(Icons.local_fire_department_rounded, color: Colors.white, size: 10),
                          ),
                          const SizedBox(width: 2),
                          // ИЗМЕНЕНИЕ: Текст дней стал чисто красным
                          Text(
                            '${streak['days']}',
                            style: TextStyle(
                              color: streak['active'] ? const Color(0xFFFF3333) : Colors.grey,
                              fontSize: 9,
                              fontWeight: FontWeight.bold
                            ),
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
                const SizedBox(height: 4),
                SizedBox(
                  width: 55,
                  child: Text(
                    streak['name'],
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                    textAlign: TextAlign.center,
                    style: TextStyle(color: Colors.white.withOpacity(0.6), fontSize: 10),
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }

  // 2. Виджет серий для ПК (Квадратные карточки и огненный градиент)
  Widget _buildPcStreakPanel() {
    return Container(
      padding: const EdgeInsets.all(16),
      decoration: BoxDecoration(
        color: const Color(0xFF17212B).withOpacity(0.6),
        borderRadius: BorderRadius.circular(20),
        border: Border.all(color: const Color(0xFF243141).withOpacity(0.3)),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        mainAxisSize: MainAxisSize.min,
        children: [
          Row(
            children: [
              // Градиентный главный огонёк в заголовке панели ПК
              ShaderMask(
                shaderCallback: (bounds) => const LinearGradient(
                  colors: [Color(0xFFFF8C00), Color(0xFFFF0000)],
                  begin: Alignment.topCenter,
                  end: Alignment.bottomCenter,
                ).createShader(bounds),
                child: const Icon(Icons.local_fire_department_rounded, color: Colors.white, size: 20),
              ),
              const SizedBox(width: 8),
              const Text(
                'Активные серии',
                style: TextStyle(color: Colors.white, fontSize: 15, fontWeight: FontWeight.bold),
              ),
            ],
          ),
          const SizedBox(height: 12),
          ..._chatStreaks.map((streak) {
            return Container(
              margin: const EdgeInsets.only(bottom: 10),
              padding: const EdgeInsets.all(10),
              decoration: BoxDecoration(
                color: const Color(0xFF1E2938).withOpacity(0.3),
                // ИЗМЕНЕНИЕ: Строгая квадратная карточка серии для ПК
                borderRadius: BorderRadius.circular(10),
              ),
              child: Row(
                children: [
                  // ИЗМЕНЕНИЕ: Квадратная аватарка вместо CircleAvatar
                  Container(
                    width: 32,
                    height: 32,
                    decoration: BoxDecoration(
                      color: const Color(0xFF243141),
                      borderRadius: BorderRadius.circular(6), // Легкое скругление углов квадрата
                    ),
                    child: Center(
                      child: Text(
                        _getInitials(streak['name']),
                        style: const TextStyle(color: Colors.white, fontSize: 11, fontWeight: FontWeight.bold)
                      ),
                    ),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    child: Text(
                      streak['name'],
                      style: const TextStyle(color: Colors.white, fontSize: 13, fontWeight: FontWeight.w500),
                    ),
                  ),
                  // Плашка со счетчиком
                  Container(
                    padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                    decoration: BoxDecoration(
                      color: const Color(0xFF0E131A),
                      borderRadius: BorderRadius.circular(6), // Тоже квадратная плашка
                    ),
                    child: Row(
                      children: [
                        // Градиентный огонёк в списке
                        ShaderMask(
                          shaderCallback: (bounds) => const LinearGradient(
                            colors: [Color(0xFFFF8C00), Color(0xFFFF0000)],
                            begin: Alignment.topCenter,
                            end: Alignment.bottomCenter,
                          ).createShader(bounds),
                          child: const Icon(Icons.local_fire_department_rounded, color: Colors.white, size: 14),
                        ),
                        const SizedBox(width: 6),
                        // ИЗМЕНЕНИЕ: Текст дней горит чисто красным
                        Text(
                          '${streak['days']} дн',
                          style: TextStyle(
                            color: streak['active'] ? const Color(0xFFFF3333) : Colors.grey,
                            fontWeight: FontWeight.bold,
                            fontSize: 12,
                          ),
                        ),
                      ],
                    ),
                  ),
                ],
              ),
            );
          }).toList(),
        ],
      ),
    );
  }



  // 3. Адаптивный виджет карточки чата
      // 3. Адаптивный виджет карточки чата
  Widget _buildChatCard(Map<String, dynamic> chat) {
    final int unreadCount = chat['unread'] ?? 0;
    final bool isMobile = MediaQuery.of(context).size.width < 600;

    return Align(
      alignment: Alignment.centerLeft,
      child: Container(
        constraints: BoxConstraints(
          maxWidth: isMobile ? double.infinity : 420,
        ),
        margin: const EdgeInsets.only(bottom: 8),
        child: Material(
          color: Colors.transparent,
          child: InkWell(
            onTap: () {
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => ChatRoomScreen(chatName: chat['name']),
                ),
              );
            },
            borderRadius: BorderRadius.circular(16),
            child: Container(
              padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 12),
              decoration: BoxDecoration(
                color: const Color(0xFF1E2938).withOpacity(0.4),
                borderRadius: BorderRadius.circular(16),
                border: Border.all(color: const Color(0xFF2F3E4E).withOpacity(0.2), width: 1),
              ),
              child: Row(
                children: [
                  Container(
                    width: 48,
                    height: 48,
                    decoration: BoxDecoration(
                      shape: BoxShape.circle,
                      color: const Color(0xFF243141).withOpacity(0.95),
                      border: Border.all(color: const Color(0xFF00F5D4).withOpacity(0.3), width: 1.5),
                    ),
                    child: Center(
                      child: Text(
                        _getInitials(chat['name']),
                        style: const TextStyle(
                          color: Color(0xFF00F5D4),
                          fontSize: 15,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ),
                  const SizedBox(width: 14),
                  Expanded(
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(
                          chat['name'],
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                          style: const TextStyle(
                            color: Colors.white,
                            fontSize: 16,
                            fontWeight: FontWeight.w600,
                          ),
                        ),
                        const SizedBox(height: 4),
                        Text(
                          chat['lastMessage'],
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                          style: TextStyle(
                            color: Colors.white.withOpacity(0.5),
                            fontSize: 13,
                          ),
                        ),
                      ],
                    ),
                  ),
                  const SizedBox(width: 8),
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.end,
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Text(
                        chat['time'],
                        style: TextStyle(
                          color: Colors.white.withOpacity(0.3),
                          fontSize: 11,
                        ),
                      ),
                      const SizedBox(height: 6),
                      if (unreadCount > 0)
                        Container(
                          padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                          decoration: BoxDecoration(
                            color: const Color(0xFF00F5D4),
                            borderRadius: BorderRadius.circular(10),
                            boxShadow: [
                              BoxShadow(
                                color: const Color(0xFF00F5D4).withOpacity(0.3),
                                blurRadius: 6,
                              ),
                            ],
                          ),
                          child: Text(
                            '$unreadCount',
                            style: const TextStyle(
                              color: Color(0xFF0E131A),
                              fontSize: 11,
                              fontWeight: FontWeight.bold,
                            ),
                          ),
                        )
                      else
                        const SizedBox(height: 19),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
