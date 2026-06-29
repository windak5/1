import 'package:flutter/material.dart';
import 'chat_room_screen.dart';

class ChatsScreen extends StatefulWidget {
  final Function(bool)? onChatToggle; // ДОБАВИТЬ ЭТУ СТРОКУ

  const ChatsScreen({super.key, this.onChatToggle}); // ОБНОВИТЬ КОНСТРУКТОР

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


  // 3. Переменные для работы поиска (теперь они строго на своем месте)
  bool _isSearching = false;
  String _searchQuery = "";
  final TextEditingController _searchController = TextEditingController();

    // Переменная для хранения имени выбранного в данный момент чата
  String? _selectedChatName;


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
    final bool isMobile = MediaQuery.of(context).size.width < 600;

    final List<Map<String, dynamic>> filteredChats = _spaceChats.where((chat) {
      final String name = chat['name'].toString().toLowerCase();
      return name.contains(_searchQuery.toLowerCase());
    }).toList();

    return Scaffold(
      backgroundColor: const Color(0xFF0E131A),
      body: SafeArea(
        child: Stack(
          children: [
            // Основная адаптивная сетка
            Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                // Шапка WinChat / Поиск
                Padding(
                  padding: const EdgeInsets.only(top: 16, left: 24, bottom: 12, right: 16),
                  child: AnimatedSwitcher(
                    duration: const Duration(milliseconds: 250),
                    child: _isSearching
                        ? Container(
                            key: const ValueKey('searchBar'),
                            height: 40,
                            decoration: BoxDecoration(
                              color: const Color(0xFF1E2938),
                              borderRadius: BorderRadius.circular(12),
                              border: Border.all(color: const Color(0xFF2F3E4E)),
                            ),
                            child: TextField(
                              controller: _searchController,
                              autofocus: true,
                              style: const TextStyle(color: Colors.white, fontSize: 15),
                              cursorColor: const Color(0xFF00F5D4),
                              onChanged: (value) => setState(() => _searchQuery = value),
                              decoration: InputDecoration(
                                hintText: 'Поиск чатов...',
                                hintStyle: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 14),
                                border: InputBorder.none,
                                prefixIcon: const Icon(Icons.search_rounded, color: Color(0xFF00F5D4), size: 20),
                                suffixIcon: IconButton(
                                  icon: const Icon(Icons.close_rounded, color: Colors.grey, size: 20),
                                  onPressed: () {
                                    setState(() {
                                      _isSearching = false;
                                      _searchQuery = '';
                                      _searchController.clear();
                                    });
                                  },
                                ),
                              ),
                            ),
                          )
                        : Row(
                            key: const ValueKey('titleBar'),
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
                ),
                if (isMobile) _buildMobileStreaks(),

                Expanded(
                  child: Row(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      // ЛЕВАЯ ПАНЕЛЬ: Список чатов (Статичная ширина flex: 4)
                      Expanded(
                        flex: 4,
                        child: Padding(
                          padding: const EdgeInsets.only(left: 24, right: 16, top: 4, bottom: 90),
                          child: filteredChats.isEmpty
                              ? Center(child: Text('Ничего не найдено', style: TextStyle(color: Colors.white.withOpacity(0.4), fontSize: 14)))
                              : Container(
                                  decoration: BoxDecoration(
                                    color: const Color(0xFF17212B).withOpacity(0.6),
                                    borderRadius: BorderRadius.circular(20),
                                    border: Border.all(color: const Color(0xFF243141).withOpacity(0.3)),
                                  ),
                                  clipBehavior: Clip.antiAlias,
                                  child: ListView.builder(
                                    padding: EdgeInsets.zero,
                                    itemCount: filteredChats.length,
                                    itemBuilder: (context, index) {
                                      final chat = filteredChats[index];
                                      final bool isLast = index == filteredChats.length - 1;

                                      return GestureDetector(
                                        onTap: () {
                                          if (isMobile) {
                                            Navigator.push(
                                              context,
                                              MaterialPageRoute(builder: (context) => ChatRoomScreen(chatName: chat['name'])),
                                            );
                                          } else {
                                            setState(() {
                                              _selectedChatName = chat['name'];
                                              if (widget.onChatToggle != null) widget.onChatToggle!(true);
                                            });
                                          }
                                        },
                                        child: _buildChatCard(chat, isLast),
                                      );
                                    },
                                  ),
                                ),
                        ),
                      ),

                      // ЦЕНТРАЛЬНАЯ ПАНЕЛЬ: Окно переписки (Занимает ровно 6 долей из 10)
                      if (!isMobile && _selectedChatName != null)
                        Expanded(
                          flex: 6,
                          child: Container(
                            color: const Color(0xFF17212B),
                            child: ChatRoomScreen(
                              chatName: _selectedChatName!,
                              onCloseOnPc: () {
                                setState(() {
                                  _selectedChatName = null;
                                  if (widget.onChatToggle != null) widget.onChatToggle!(false);
                                });
                              },
                            ),
                          ),
                        ),

                      // ИСПРАВЛЕНИЕ ТУТ: ПРАВАЯ ЧАСТЬ ЭКРАНА (flex: 6), когда чат закрыт.
                      // Центрирует плашку серий строго по центру оставшейся области.
                      if (!isMobile && _selectedChatName == null)
                        Expanded(
                          flex: 6,
                          child: Center( // Центрирует содержимое и по горизонтали, и по вертикали
                            child: Padding(
                              padding: const EdgeInsets.only(bottom: 90), // Сбалансированный отступ под нижний остров
                              child: SizedBox(
                                width: 440, // Идеальная фиксированная ширина для аккуратного квадрата
                                child: _buildPcStreakPanel(),
                              ),
                            ),
                          ),
                        ),
                    ],
                  ),
                ),
              ],
            ),

            // Кнопки поиска и группы (показываются только когда чат закрыт)
            if (_selectedChatName == null)
              _buildActionButtons(),
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
      // ИСПРАВЛЕНИЕ: Жестко задаем одинаковую ширину и высоту для идеального квадрата
      width: 340,
      height: 340,
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
              // Защита заголовка от переполнения
              const Expanded(
                child: Text(
                  'Активные серии',
                  maxLines: 1,
                  overflow: TextOverflow.ellipsis,
                  style: TextStyle(color: Colors.white, fontSize: 15, fontWeight: FontWeight.bold),
                ),
              ),
            ],
          ),
          const SizedBox(height: 12),
          // Оборачиваем список в Expanded, чтобы он аккуратно занимал место внутри квадрата
          Expanded(
            child: ListView(
              padding: EdgeInsets.zero,
              physics: const NeverScrollableScrollPhysics(), // Отключаем внутренний скролл
              children: _chatStreaks.map((streak) {
                return Container(
                  margin: const EdgeInsets.only(bottom: 10),
                  padding: const EdgeInsets.all(10),
                  decoration: BoxDecoration(
                    color: const Color(0xFF1E2938).withOpacity(0.3),
                    borderRadius: BorderRadius.circular(10),
                  ),
                  child: Row(
                    children: [
                      // Квадратная аватарка с легким скруглением
                      Container(
                        width: 32,
                        height: 32,
                        decoration: BoxDecoration(
                          color: const Color(0xFF243141),
                          borderRadius: BorderRadius.circular(6),
                        ),
                        child: Center(
                          child: Text(
                            _getInitials(streak['name']),
                            style: const TextStyle(color: Colors.white, fontSize: 11, fontWeight: FontWeight.bold)
                          ),
                        ),
                      ),
                      const SizedBox(width: 12),
                      // Имя плавно уходит в троеточие
                      Expanded(
                        child: Text(
                          streak['name'],
                          maxLines: 1,
                          overflow: TextOverflow.ellipsis,
                          style: const TextStyle(color: Colors.white, fontSize: 13, fontWeight: FontWeight.w500),
                        ),
                      ),
                      const SizedBox(width: 8),
                      // Плашка со счетчиком дней
                      Container(
                        padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                        decoration: BoxDecoration(
                          color: const Color(0xFF0E131A),
                          borderRadius: BorderRadius.circular(6),
                        ),
                        child: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            ShaderMask(
                              shaderCallback: (bounds) => const LinearGradient(
                                colors: [Color(0xFFFF8C00), Color(0xFFFF0000)],
                                begin: Alignment.topCenter,
                                end: Alignment.bottomCenter,
                              ).createShader(bounds),
                              child: const Icon(Icons.local_fire_department_rounded, color: Colors.white, size: 14),
                            ),
                            const SizedBox(width: 6),
                            // Текст дней
                            Text(
                              '${streak['days']} дн',
                              style: TextStyle(
                                color: streak['active'] ? const Color(0xFFFF9F1C) : Colors.grey,
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
            ),
          ),
        ],
      ),
    );
  }

  // 3. Адаптивный виджет карточки чата
     Widget _buildChatCard(Map<String, dynamic> chat, bool isLast) {
    final int unreadCount = chat['unread'] ?? 0;

    return Material(
      color: Colors.transparent,
      child: Column(
        children: [
          Container(
            padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 12),
            child: Row(
              children: [
                // Аватарка чата
                Container(
                  width: 48, height: 48,
                  decoration: const BoxDecoration(color: Color(0xFF2F3E4E), shape: BoxShape.circle),
                  child: Center(
                    child: Text(
                      _getInitials(chat['name']),
                      style: const TextStyle(color: Color(0xFF00F5D4), fontWeight: FontWeight.bold, fontSize: 16),
                    ),
                  ),
                ),
                const SizedBox(width: 12),
                // Информационный блок (Имя, Время, Сообщение)
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Expanded(
                            child: Text(
                              chat['name'],
                              maxLines: 1, overflow: TextOverflow.ellipsis,
                              style: const TextStyle(color: Colors.white, fontSize: 15, fontWeight: FontWeight.bold),
                            ),
                          ),
                          const SizedBox(width: 8),
                          Text(chat['time'], style: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 12)),
                        ],
                      ),
                      const SizedBox(height: 4),
                      Row(
                        mainAxisAlignment: MainAxisAlignment.spaceBetween,
                        children: [
                          Expanded(
                            child: Text(
                              chat['lastMessage'],
                              maxLines: 1, overflow: TextOverflow.ellipsis,
                              style: TextStyle(color: Colors.white.withOpacity(0.4), fontSize: 13),
                            ),
                          ),
                          if (unreadCount > 0) ...[
                            const SizedBox(width: 8),
                            Container(
                              padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
                              decoration: BoxDecoration(color: const Color(0xFF00F5D4), borderRadius: BorderRadius.circular(10)),
                              child: Text(
                                unreadCount.toString(),
                                style: const TextStyle(color: Color(0xFF0E131A), fontSize: 11, fontWeight: FontWeight.bold),
                              ),
                            ),
                          ],
                        ],
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
          // Тонкая линия-разделитель (не показываем для самого последнего чата в списке)
          if (!isLast)
            Padding(
              padding: const EdgeInsets.only(left: 76, right: 16), // Смещение, чтобы линия начиналась от текста, а не под аватаркой
              child: Divider(color: const Color(0xFF2F3E4E).withOpacity(0.3), height: 1, thickness: 1),
            ),
        ],
      ),
    );
  }

// Кнопки управления поиском и созданием групп
Widget _buildActionButtons() {
  return Positioned(
    right: 24,
    bottom: 24,
    child: Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // Серая круглая кнопка создания группы
        GestureDetector(
          onTap: () => _showCreateGroupDialog(context),
          child: Container(
            width: 56, height: 56,
            decoration: const BoxDecoration(color: Color(0xFF2F3E4E), shape: BoxShape.circle),
            child: const Icon(Icons.group_add_rounded, color: Colors.white, size: 24),
          ),
        ),
        const SizedBox(height: 12),
        // Фирменная капля-поиск
        GestureDetector(
          onTap: () {
            setState(() {
              _isSearching = !_isSearching;
              if (!_isSearching) {
                _searchQuery = '';
                _searchController.clear();
              }
            });
          },
          child: Container(
            width: 56, height: 56,
            decoration: const BoxDecoration(
              color: Color(0xFF00F5D4),
              borderRadius: BorderRadius.only(
                topLeft: Radius.circular(28),
                topRight: Radius.circular(28),
                bottomLeft: Radius.circular(28),
                bottomRight: Radius.circular(4),
              ),
            ),
            child: Icon(
              _isSearching ? Icons.search_off_rounded : Icons.search_rounded,
              color: const Color(0xFF0E131A),
              size: 26,
            ),
          ),
        ),
      ],
    ),
  );
}
// Метод вызова окна для добавления нового чата/группы
  // Обновленный метод вызова окна для создания ГРУППЫ
  void _showCreateGroupDialog(BuildContext context) {
    final TextEditingController nameController = TextEditingController();
    final TextEditingController usersController = TextEditingController(); // Контроллер для участников

    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          backgroundColor: const Color(0xFF1E2938),
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
          title: const Row(
            children: [
              Icon(Icons.group_add_rounded, color: Color(0xFF00F5D4), size: 22),
              SizedBox(width: 8),
              Text('Создать группу', style: TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold)),
            ],
          ),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              // Поле 1: Название группы
              TextField(
                controller: nameController,
                autofocus: true,
                style: const TextStyle(color: Colors.white),
                cursorColor: const Color(0xFF00F5D4),
                decoration: InputDecoration(
                  hintText: 'Название группы',
                  hintStyle: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 14),
                  enabledBorder: const UnderlineInputBorder(borderSide: BorderSide(color: Color(0xFF2F3E4E))),
                  focusedBorder: const UnderlineInputBorder(borderSide: BorderSide(color: Color(0xFF00F5D4))),
                ),
              ),
              const SizedBox(height: 16),
              // Поле 2: Участники
              TextField(
                controller: usersController,
                style: const TextStyle(color: Colors.white),
                cursorColor: const Color(0xFF00F5D4),
                decoration: InputDecoration(
                  hintText: 'Введите никнеймы через запятую',
                  hintStyle: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 13),
                  enabledBorder: const UnderlineInputBorder(borderSide: BorderSide(color: Color(0xFF2F3E4E))),
                  focusedBorder: const UnderlineInputBorder(borderSide: BorderSide(color: Color(0xFF00F5D4))),
                ),
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: Text('Отмена', style: TextStyle(color: Colors.white.withOpacity(0.5))),
            ),
            TextButton(
              onPressed: () {
                if (nameController.text.trim().isNotEmpty) {
                  setState(() {
                    // Добавляем созданную группу в ваш список чатов
                    _spaceChats.insert(0, {
                      'name': nameController.text.trim(),
                      'lastMessage': 'Группа создана. Участники: ${usersController.text.isEmpty ? "нет" : usersController.text.trim()}',
                      'unread': 0,
                      'time': 'Только что',
                      'isGroup': true, // Добавляем флаг, что это именно группа!
                    });
                  });
                  Navigator.pop(context);
                }
              },
              child: const Text('Создать', style: TextStyle(color: Color(0xFF00F5D4), fontWeight: FontWeight.bold)),
            ),
          ],
        );
      },
    );
  }

 // Центральная панель: заглушка или окно сообщений
Widget _buildCentralMessagePanel() {
  if (_selectedChatName == null) {
    // 1. Состояние: Чат не выбран (Заглушка)
    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          Icon(Icons.forum_rounded, size: 64, color: const Color(0xFF00F5D4).withOpacity(0.2)),
          const SizedBox(height: 16),
          Text(
            'Выберите чат, чтобы начать общение',
            style: TextStyle(color: Colors.white.withOpacity(0.4), fontSize: 15),
          ),
        ],
      ),
    );
  }

  // 2. Состояние: Чат выбран (Окно переписки)
  return Container(
    decoration: const BoxDecoration(
      border: Border(
        left: BorderSide(color: Color(0xFF1E2938), width: 1),
        right: BorderSide(color: Color(0xFF1E2938), width: 1),
      ),
    ),
    child: Column(
      children: [
        // Шапка открытого чата
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 14),
          color: const Color(0xFF111823),
          child: Row(
            children: [
              CircleAvatar(
                backgroundColor: const Color(0xFF1E2938),
                radius: 18,
                child: Text(_selectedChatName![0], style: const TextStyle(color: Color(0xFF00F5D4), fontWeight: FontWeight.bold)),
              ),
              const SizedBox(width: 12),
              Text(_selectedChatName!, style: const TextStyle(color: Colors.white, fontSize: 16, fontWeight: FontWeight.w600)),
            ],
          ),
        ),
        // Область сообщений (Фейковая история для демонстрации)
        Expanded(
          child: ListView(
            padding: const EdgeInsets.all(20),
            children: [
              _buildDemoBubble('Привет! Как продвигается разработка WinChat?', false),
              _buildDemoBubble('Привет) Да вот, настраиваю центральную панель сообщений и адаптивный интерфейс.', true),
              _buildDemoBubble('Выглядит очень круто, цвета огонь! 🔥', false),
            ],
          ),
        ),
        // Поле ввода сообщения внизу чата
        Padding(
          padding: const EdgeInsets.only(left: 16, right: 16, bottom: 20),
          child: Container(
            decoration: BoxDecoration(color: const Color(0xFF1E2938), borderRadius: BorderRadius.circular(12)),
            child: TextField(
              style: const TextStyle(color: Colors.white, fontSize: 14),
              cursorColor: const Color(0xFF00F5D4),
              decoration: InputDecoration(
                hintText: 'Напишите сообщение...',
                hintStyle: TextStyle(color: Colors.white.withOpacity(0.3), fontSize: 14),
                border: InputBorder.none,
                contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
                suffixIcon: const Icon(Icons.send_rounded, color: Color(0xFF00F5D4), size: 20),
              ),
            ),
          ),
        ),
      ],
    ),
  );
}

// Вспомогательный виджет для красивых фейковых облачков сообщений
Widget _buildDemoBubble(String text, bool isMe) {
  return Align(
    alignment: isMe ? Alignment.centerRight : Alignment.centerLeft,
    child: Container(
      margin: const EdgeInsets.symmetric(vertical: 6),
      padding: const EdgeInsets.symmetric(horizontal: 14, vertical: 10),
      constraints: const BoxConstraints(maxWidth: 400),
      decoration: BoxDecoration(
        color: isMe ? const Color(0xFF00F5D4).withOpacity(0.15) : const Color(0xFF1E2938),
        borderRadius: BorderRadius.only(
          topLeft: const Radius.circular(12), topRight: const Radius.circular(12),
          bottomLeft: Radius.circular(isMe ? 12 : 2), bottomRight: Radius.circular(isMe ? 2 : 12),
        ),
        border: isMe ? Border.all(color: const Color(0xFF00F5D4).withOpacity(0.3)) : null,
      ),
      child: Text(text, style: TextStyle(color: isMe ? const Color(0xFF00F5D4) : Colors.white, fontSize: 14)),
    ),
  );
}
}
