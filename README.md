import 'dart:math' as math;
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  runApp(const CalculatorApp());
}

class CalculatorApp extends StatelessWidget {
  const CalculatorApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Calculator',
      theme: ThemeData(
        brightness: Brightness.dark,
        scaffoldBackgroundColor: const Color(0xFF0B0D10),
        useMaterial3: true,
      ),
      home: const CalculatorPage(),
    );
  }
}

enum CalcButtonType {
  number,
  operator,
  function,
  action,
  memory,
}

class CalcButtonData {
  final String label;
  final String action;
  final CalcButtonType type;

  const CalcButtonData(
    this.label,
    this.action, {
    this.type = CalcButtonType.number,
  });
}

class CalculatorPage extends StatefulWidget {
  const CalculatorPage({super.key});

  @override
  State<CalculatorPage> createState() => _CalculatorPageState();
}

class _CalculatorPageState extends State<CalculatorPage> {
  String expression = '';
  String result = '0';

  double ans = 0;
  double memory = 0;

  bool shift = false;
  bool second = false;
  bool degrees = true;

  final List<String> history = [];

  static const int maxExpressionLength = 1000;

  final ScrollController _displayScrollController = ScrollController();

  @override
  void dispose() {
    _displayScrollController.dispose();
    super.dispose();
  }

  void insert(String value) {
    if (expression.length >= maxExpressionLength) return;

    setState(() {
      expression += value;

      if (expression.length > maxExpressionLength) {
        expression = expression.substring(0, maxExpressionLength);
      }
    });

    _scrollDisplayToEnd();
  }

  void _scrollDisplayToEnd() {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (_displayScrollController.hasClients) {
        _displayScrollController.animateTo(
          _displayScrollController.position.maxScrollExtent,
          duration: const Duration(milliseconds: 100),
          curve: Curves.easeOut,
        );
      }
    });
  }

  void backspace() {
    if (expression.isEmpty) return;

    setState(() {
      expression = expression.substring(0, expression.length - 1);
    });
  }

  void clearAll() {
    setState(() {
      expression = '';
      result = '0';
    });
  }

  void evaluateExpression() {
    if (expression.trim().isEmpty) return;

    try {
      // The display uses modern calculator symbols:
      // × ÷ −
      // Internally the parser uses:
      // * / -
      final normalized = expression
          .replaceAll('×', '*')
          .replaceAll('÷', '/')
          .replaceAll('−', '-');

      final parser = ExpressionParser(
        normalized,
        ans: ans,
        degrees: degrees,
      );

      final value = parser.parse();

      if (value.isNaN || value.isInfinite) {
        throw const FormatException('Invalid result');
      }

      ans = value;

      final formatted = formatNumber(value);

      setState(() {
        result = formatted;

        history.insert(
          0,
          '$expression = $formatted',
        );

        if (history.length > 50) {
          history.removeLast();
        }
      });
    } catch (_) {
      setState(() {
        result = 'Error';
      });
    }
  }

  String formatNumber(double value) {
    if (value.abs() < 1e-12) {
      value = 0;
    }

    if (value == value.roundToDouble()) {
      return value.toInt().toString();
    }

    final abs = value.abs();

    if (abs >= 1e12 || (abs > 0 && abs < 1e-10)) {
      return value
          .toStringAsExponential(10)
          .replaceFirst(RegExp(r'0+$'), '')
          .replaceFirst(RegExp(r'\.$'), '');
    }

    String text = value.toStringAsPrecision(14);

    if (text.contains('e')) {
      return text;
    }

    text = text.replaceFirst(RegExp(r'0+$'), '');
    text = text.replaceFirst(RegExp(r'\.$'), '');

    return text;
  }

  void handleKey(String key) {
    switch (key) {
      case 'AC':
        clearAll();
        break;

      case '⌫':
        backspace();
        break;

      case '=':
        evaluateExpression();
        break;

      case 'SHIFT':
        setState(() {
          shift = !shift;
        });
        break;

      case '2nd':
        setState(() {
          second = !second;
        });
        break;

      case 'DEG':
        setState(() {
          degrees = true;
        });
        break;

      case 'RAD':
        setState(() {
          degrees = false;
        });
        break;

      case 'M+':
        _memoryAdd();
        break;

      case 'M-':
        _memorySubtract();
        break;

      case 'RCL':
        insert(formatNumber(memory));
        break;

      case 'MC':
        setState(() {
          memory = 0;
        });
        break;

      case 'COPY':
        Clipboard.setData(
          ClipboardData(
            text: result,
          ),
        );
        break;

      case 'HISTORY':
        _showHistory();
        break;

      case 'ANS':
        insert('Ans');
        break;

      case 'π':
        insert('π');
        break;

      case 'e':
        insert('e');
        break;

      case 'EXP':
        insert('E');
        break;

      case 'x²':
        insert('^2');
        break;

      case 'x³':
        insert('^3');
        break;

      case 'xʸ':
        insert('^');
        break;

      case '√':
        insert('sqrt(');
        break;

      case '∛':
        insert('cbrt(');
        break;

      case 'log':
        insert('log(');
        break;

      case 'ln':
        insert('ln(');
        break;

      case 'sin':
        insert('sin(');
        break;

      case 'cos':
        insert('cos(');
        break;

      case 'tan':
        insert('tan(');
        break;

      case 'asin':
        insert('asin(');
        break;

      case 'acos':
        insert('acos(');
        break;

      case 'atan':
        insert('atan(');
        break;

      case 'sinh':
        insert('sinh(');
        break;

      case 'cosh':
        insert('cosh(');
        break;

      case 'tanh':
        insert('tanh(');
        break;

      case 'asinh':
        insert('asinh(');
        break;

      case 'acosh':
        insert('acosh(');
        break;

      case 'atanh':
        insert('atanh(');
        break;

      case 'abs':
        insert('abs(');
        break;

      case 'floor':
        insert('floor(');
        break;

      case 'ceil':
        insert('ceil(');
        break;

      case 'fact':
        insert('!');
        break;

      case 'nPr':
        insert('nPr(');
        break;

      case 'nCr':
        insert('nCr(');
        break;

      case 'gcd':
        insert('gcd(');
        break;

      case 'lcm':
        insert('lcm(');
        break;

      case 'mod':
        insert('mod(');
        break;

      case '%':
        insert('%');
        break;

      default:
        insert(key);
    }
  }

  void _memoryAdd() {
    final value = _currentNumericValue();
    if (value == null) return;

    setState(() {
      memory += value;
    });
  }

  void _memorySubtract() {
    final value = _currentNumericValue();
    if (value == null) return;

    setState(() {
      memory -= value;
    });
  }

  double? _currentNumericValue() {
    try {
      final source = expression.isNotEmpty ? expression : result;

      final normalized = source
          .replaceAll('×', '*')
          .replaceAll('÷', '/')
          .replaceAll('−', '-');

      return ExpressionParser(
        normalized,
        ans: ans,
        degrees: degrees,
      ).parse();
    } catch (_) {
      return null;
    }
  }

  void _showHistory() {
    showModalBottomSheet(
      context: context,
      backgroundColor: const Color(0xFF15181D),
      isScrollControlled: true,
      builder: (context) {
        return SafeArea(
          child: SizedBox(
            height: MediaQuery.of(context).size.height * 0.7,
            child: Column(
              children: [
                const SizedBox(height: 16),
                const Text(
                  'History',
                  style: TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 12),
                Expanded(
                  child: history.isEmpty
                      ? const Center(
                          child: Text(
                            'No calculations yet',
                            style: TextStyle(
                              color: Colors.white54,
                            ),
                          ),
                        )
                      : ListView.separated(
                          padding: const EdgeInsets.all(16),
                          itemCount: history.length,
                          separatorBuilder: (_, __) =>
                              const Divider(height: 1),
                          itemBuilder: (context, index) {
                            return ListTile(
                              title: Text(
                                history[index],
                                textAlign: TextAlign.right,
                              ),
                              onTap: () {
                                Navigator.pop(context);
                                final item = history[index];
                                final parts = item.split(' = ');

                                if (parts.isNotEmpty) {
                                  setState(() {
                                    expression = parts.first;
                                    if (parts.length > 1) {
                                      result = parts[1];
                                    }
                                  });
                                }
                              },
                            );
                          },
                        ),
                ),
                if (history.isNotEmpty)
                  Padding(
                    padding: const EdgeInsets.all(16),
                    child: SizedBox(
                      width: double.infinity,
                      child: FilledButton(
                        onPressed: () {
                          setState(() {
                            history.clear();
                          });
                          Navigator.pop(context);
                        },
                        child: const Text('Clear History'),
                      ),
                    ),
                  ),
              ],
            ),
          ),
        );
      },
    );
  }

  List<CalcButtonData> get buttons {
    final List<CalcButtonData> list = [];

    list.addAll([
      const CalcButtonData(
        'SHIFT',
        'SHIFT',
        type: CalcButtonType.action,
      ),
      const CalcButtonData(
        '2nd',
        '2nd',
        type: CalcButtonType.action,
      ),
      CalcButtonData(
        degrees ? 'DEG' : 'RAD',
        degrees ? 'RAD' : 'DEG',
        type: CalcButtonType.action,
      ),
      const CalcButtonData(
        'HIST',
        'HISTORY',
        type: CalcButtonType.action,
      ),
      const CalcButtonData(
        '⌫',
        '⌫',
        type: CalcButtonType.action,
      ),
      const CalcButtonData(
        'AC',
        'AC',
        type: CalcButtonType.action,
      ),
    ]);

    list.addAll([
      const CalcButtonData(
        'sin',
        'sin',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'cos',
        'cos',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'tan',
        'tan',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'log',
        'log',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'ln',
        'ln',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        '√',
        '√',
        type: CalcButtonType.function,
      ),
    ]);

    list.addAll([
      const CalcButtonData(
        '(',
        '(',
        type: CalcButtonType.operator,
      ),
      const CalcButtonData(
        ')',
        ')',
        type: CalcButtonType.operator,
      ),
      const CalcButtonData(
        'π',
        'π',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'e',
        'e',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'x²',
        'x²',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'xʸ',
        'xʸ',
        type: CalcButtonType.function,
      ),
    ]);

    list.addAll([
      const CalcButtonData('7', '7'),
      const CalcButtonData('8', '8'),
      const CalcButtonData('9', '9'),
      const CalcButtonData(
        '÷',
        '÷',
        type: CalcButtonType.operator,
      ),
      const CalcButtonData(
        'mod',
        'mod',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        '%',
        '%',
        type: CalcButtonType.operator,
      ),
    ]);

    list.addAll([
      const CalcButtonData('4', '4'),
      const CalcButtonData('5', '5'),
      const CalcButtonData('6', '6'),
      const CalcButtonData(
        '×',
        '×',
        type: CalcButtonType.operator,
      ),
      const CalcButtonData(
        '−',
        '−',
        type: CalcButtonType.operator,
      ),
      const CalcButtonData(
        '+',
        '+',
        type: CalcButtonType.operator,
      ),
    ]);

    list.addAll([
      const CalcButtonData('1', '1'),
      const CalcButtonData('2', '2'),
      const CalcButtonData('3', '3'),
      const CalcButtonData(
        'Ans',
        'ANS',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        'EXP',
        'EXP',
        type: CalcButtonType.function,
      ),
      const CalcButtonData(
        '=',
        '=',
        type: CalcButtonType.operator,
      ),
    ]);

    list.addAll([
      const CalcButtonData('0', '0'),
      const CalcButtonData('.', '.'),
      const CalcButtonData(
        'M+',
        'M+',
        type: CalcButtonType.memory,
      ),
      const CalcButtonData(
        'M−',
        'M-',
        type: CalcButtonType.memory,
      ),
      const CalcButtonData(
        'RCL',
        'RCL',
        type: CalcButtonType.memory,
      ),
      const CalcButtonData(
        'MC',
        'MC',
        type: CalcButtonType.memory,
      ),
    ]);

    return list;
  }

  String get secondaryFunctionLabel {
    if (!shift && !second) {
      return 'Scientific';
    }

    if (shift && !second) {
      return 'Inverse / Hyperbolic';
    }

    if (second) {
      return 'Advanced';
    }

    return 'Scientific';
  }

  List<CalcButtonData> get scientificButtons {
    if (second) {
      return const [
        CalcButtonData(
          'asin',
          'asin',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'acos',
          'acos',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'atan',
          'atan',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'abs',
          'abs',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'floor',
          'floor',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'ceil',
          'ceil',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'fact',
          'fact',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'nPr',
          'nPr',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'nCr',
          'nCr',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'gcd',
          'gcd',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'lcm',
          'lcm',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'cbrt',
          'cbrt(',
          type: CalcButtonType.function,
        ),
      ];
    }

    if (shift) {
      return const [
        CalcButtonData(
          'sinh',
          'sinh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'cosh',
          'cosh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'tanh',
          'tanh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'asinh',
          'asinh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'acosh',
          'acosh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'atanh',
          'atanh',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          '∛',
          '∛',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'x³',
          'x³',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'exp',
          'exp(',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'floor',
          'floor',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'ceil',
          'ceil',
          type: CalcButtonType.function,
        ),
        CalcButtonData(
          'abs',
          'abs',
          type: CalcButtonType.function,
        ),
      ];
    }

    return const [
      CalcButtonData(
        'sin',
        'sin',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'cos',
        'cos',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'tan',
        'tan',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'log',
        'log',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'ln',
        'ln',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        '√',
        '√',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'x²',
        'x²',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'xʸ',
        'xʸ',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'π',
        'π',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'e',
        'e',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'nPr',
        'nPr',
        type: CalcButtonType.function,
      ),
      CalcButtonData(
        'nCr',
        'nCr',
        type: CalcButtonType.function,
      ),
    ];
  }

  @override
  Widget build(BuildContext context) {
    final screenWidth = MediaQuery.of(context).size.width;

    return Scaffold(
      body: SafeArea(
        child: Column(
          children: [
            _buildTopBar(),
            _buildDisplay(),
            _buildScientificPanel(),
            Expanded(
              child: _buildKeypad(screenWidth),
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildTopBar() {
    return Padding(
      padding: const EdgeInsets.fromLTRB(14, 8, 14, 6),
      child: Row(
        children: [
          const Expanded(
            child: Text(
              'Scientific Calculator',
              style: TextStyle(
                fontSize: 17,
                fontWeight: FontWeight.w600,
              ),
            ),
          ),
          Text(
            'M: ${formatNumber(memory)}',
            style: const TextStyle(
              fontSize: 12,
              co# Clock
Flutter project created by KLENCOD IDE
