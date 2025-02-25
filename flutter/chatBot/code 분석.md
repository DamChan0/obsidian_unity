# chatBot Frontend
```dart
	// main.dart
	import 'package:flutter/material.dart';
	import 'package:http/http.dart' as http;
	import 'dart:convert';
	
	void main() {
	  runApp(ChatApp());
	}
	
	Map<int, Color> colorShades = {
	  50: Color.fromRGBO(120, 15, 218, .1),
	  100: Color.fromRGBO(120, 15, 218, .2),
	  200: Color.fromRGBO(120, 15, 218, .3),
	  300: Color.fromRGBO(120, 15, 218, .4),
	  400: Color.fromRGBO(120, 15, 218, .5),
	  500: Color.fromRGBO(120, 15, 218, .6),
	  600: Color.fromRGBO(120, 15, 218, .7),
	  700: Color.fromRGBO(120, 15, 218, .8),
	  800: Color.fromRGBO(120, 15, 218, .9),
	  900: Color.fromRGBO(120, 15, 218, 1),
	};
	
	MaterialColor customPurple = MaterialColor(0xFF780FDA, colorShades);
	class ChatApp extends StatelessWidget {
	  @override
	  Widget build(BuildContext context) {
	    return MaterialApp(
	      title: 'JChat',
	      theme: ThemeData(
	        primarySwatch: customPurple, // ✅ 올바른 MaterialColor 적용
	      ),
	      home: ChatScreen(),
	    );
	  }
	}
	
	class ChatScreen extends StatefulWidget {
	  @override
	  _ChatScreenState createState() => _ChatScreenState();
	}
	
	class _ChatScreenState extends State<ChatScreen> {
	  final TextEditingController _controller = TextEditingController();
	  final List<Map<String, String>> _messages = [];
	  bool _isLoading = false;
	
	  // 백엔드 서버 URL (환경에 맞게 수정 필요)
	  final String _apiUrl = 'http://192.168.45.122:8080/chat';
	
	  Future<void> _sendMessage(String message) async {
	    if (message.trim().isEmpty) return;
	
	    setState(() {
	      _messages.add({'role': 'user', 'content': message});
	      _isLoading = true;
	    });
	
	    try {
	      final response = await http.post(
	        Uri.parse(_apiUrl),
	        headers: {'Content-Type': 'application/json'},
	        body: jsonEncode({'message': message}),
	      );
	
	      print('Response body: ${response.body}');
	      if (response.statusCode == 200) {
	        final data = jsonDecode(response.body);
	        print('Parsed data: $data');
	        setState(() {
	          _messages.add({'role': 'gpt', 'content': data['response']});
	        });
	      } else {
	        setState(() {
	          _messages.add(
	              {'role': 'error', 'content': 'Error: ${response.statusCode}'});
	        });
	      }
	    } catch (e) {
	      setState(() {
	        _messages.add({'role': 'error', 'content': 'Error: $e'});
	      });
	    } finally {
	      setState(() {
	        _isLoading = false;
	      });
	      _controller.clear();
	    }
	  }
	
	  @override
	  Widget build(BuildContext context) {
	    return Scaffold(
	      appBar: AppBar(
	        title: Text('JChat'),
	      ),
	      body: Column(
	        children: [
	          Expanded(
	            child: ListView.builder(
	              padding: EdgeInsets.all(8.0),
	              itemCount: _messages.length,
	              itemBuilder: (context, index) {
	                final message = _messages[index];
	                final isUser = message['role'] == 'user';
	                return Align(
	                  alignment:
	                      isUser ? Alignment.centerRight : Alignment.centerLeft,
	                  child: Container(
	                    margin: EdgeInsets.symmetric(vertical: 4.0),
	                    padding: EdgeInsets.all(12.0),
	                    decoration: BoxDecoration(
	                      color: isUser ? Colors.blue[100] : Colors.grey[200],
	                      borderRadius: BorderRadius.circular(8.0),
	                    ),
	                    child: Text(
	                      message['content']!,
	                      style: TextStyle(
	                        color: isUser ? Colors.black87 : Colors.black54,
	                      ),
	                    ),
	                  ),
	                );
	              },
	            ),
	          ),
	          if (_isLoading)
	            Padding(
	              padding: EdgeInsets.all(8.0),
	              child: CircularProgressIndicator(),
	            ),
	          Padding(
	            padding: EdgeInsets.all(8.0),
	            child: Row(
	              children: [
	                Expanded(
	                  child: TextField(
	                    controller: _controller,
	                    decoration: InputDecoration(
	                      hintText: 'Type your message...',
	                      border: OutlineInputBorder(
	                        borderRadius: BorderRadius.circular(8.0),
	                      ),
	                    ),
	                    onSubmitted: _sendMessage,
	                  ),
	                ),
	                SizedBox(width: 8.0),
	                IconButton(
	                  icon: Icon(Icons.send),
	                  onPressed: () => _sendMessage(_controller.text),
	                ),
	              ],
	            ),
	          ),
	        ],
	      ),
	    );
	  }
	}
```
