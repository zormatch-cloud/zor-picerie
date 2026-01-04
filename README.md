dependencies:
  flutter:
    sdk: flutter
  shared_preferences: ^2.2.2  # أضف هذا السطر
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'dart:convert'; // ضروري لتحويل البيانات لنصوص

void main() {
  runApp(ZorEpicerieApp());
}

class ZorEpicerieApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Zor Épicerie',
      theme: ThemeData(primarySwatch: Colors.teal),
      home: HomePage(),
    );
  }
}

// نموذج البيانات مع ميزات التحويل لـ JSON
class Product {
  String name;
  String category;
  double wholesalePrice;
  double retailPrice;
  int quantity;

  Product({
    required this.name,
    required this.category,
    required this.wholesalePrice,
    required this.retailPrice,
    required this.quantity,
  });

  // تحويل الكائن إلى خريطة (Map) للحفظ
  Map<String, dynamic> toMap() {
    return {
      'name': name,
      'category': category,
      'wholesalePrice': wholesalePrice,
      'retailPrice': retailPrice,
      'quantity': quantity,
    };
  }

  // استعادة الكائن من خريطة (Map)
  factory Product.fromMap(Map<String, dynamic> map) {
    return Product(
      name: map['name'],
      category: map['category'],
      wholesalePrice: map['wholesalePrice'],
      retailPrice: map['retailPrice'],
      quantity: map['quantity'],
    );
  }
}

class HomePage extends StatefulWidget {
  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<Product> inventory = [];
  
  final nameController = TextEditingController();
  final categoryController = TextEditingController();
  final wholesaleController = TextEditingController();
  final retailController = TextEditingController();
  final qtyController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _loadData(); // تحميل البيانات فور فتح التطبيق
  }

  // وظيفة حفظ البيانات في ذاكرة الهاتف
  Future<void> _saveData() async {
    final prefs = await SharedPreferences.getInstance();
    List<String> jsonList = inventory.map((item) => jsonEncode(item.toMap())).toList();
    await prefs.setStringList('my_inventory', jsonList);
  }

  // وظيفة تحميل البيانات من ذاكرة الهاتف
  Future<void> _loadData() async {
    final prefs = await SharedPreferences.getInstance();
    List<String>? jsonList = prefs.getStringList('my_inventory');
    if (jsonList != null) {
      setState(() {
        inventory = jsonList.map((item) => Product.fromMap(jsonDecode(item))).toList();
      });
    }
  }

  void _addProduct() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('إضافة سلعة جديدة', textAlign: TextAlign.right),
        content: SingleChildScrollView(
          child: Column(
            children: [
              TextField(controller: nameController, decoration: InputDecoration(labelText: 'اسم السلعة')),
              TextField(controller: categoryController, decoration: InputDecoration(labelText: 'الصنف')),
              TextField(controller: wholesaleController, decoration: InputDecoration(labelText: 'ثمن الجملة'), keyboardType: TextInputType.number),
              TextField(controller: retailController, decoration: InputDecoration(labelText: 'ثمن التقسيط'), keyboardType: TextInputType.number),
              TextField(controller: qtyController, decoration: InputDecoration(labelText: 'الكمية الحالية'), keyboardType: TextInputType.number),
            ],
          ),
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: Text('إلغاء')),
          ElevatedButton(
            onPressed: () {
              setState(() {
                inventory.add(Product(
                  name: nameController.text,
                  category: categoryController.text,
                  wholesalePrice: double.tryParse(wholesaleController.text) ?? 0,
                  retailPrice: double.tryParse(retailController.text) ?? 0,
                  quantity: int.tryParse(qtyController.text) ?? 0,
                ));
              });
              _saveData(); // حفظ بعد الإضافة
              Navigator.pop(context);
              // مسح الحقول
              nameController.clear(); categoryController.clear();
              wholesaleController.clear(); retailController.clear(); qtyController.clear();
            },
            child: Text('إضافة'),
          ),
        ],
      ),
    );
  }

  void _deleteProduct(int index) {
    setState(() {
      inventory.removeAt(index);
    });
    _saveData(); // حفظ بعد الحذف
  }

  @override
  Widget build(BuildContext context) {
    return Directionality(
      textDirection: TextDirection.rtl,
      child: Scaffold(
        appBar: AppBar(
          title: Text('Zor Épicerie 🛒'),
          centerTitle: true,
          actions: [
            // أيقونة لعرض إجمالي الأرباح المحتملة
            IconButton(
              icon: Icon(Icons.analytics),
              onPressed: () {
                double totalProfit = inventory.fold(0, (sum, item) => sum + ((item.retailPrice - item.wholesalePrice) * item.quantity));
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('إجمالي الأرباح المتوقعة في المخزن: $totalProfit دج')),
                );
              },
            )
          ],
        ),
        body: inventory.isEmpty
            ? Center(child: Text('المخزن فارغ، ابدأ بإضافة سلع!'))
            : ListView.builder(
                itemCount: inventory.length,
                itemBuilder: (context, index) {
                  final item = inventory[index];
                  return Card(
                    elevation: 3,
                    margin: EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                    child: ListTile(
                      title: Text(item.name, style: TextStyle(fontWeight: FontWeight.bold, color: Colors.teal)),
                      subtitle: Text('الصنف: ${item.category} | الكمية: ${item.quantity}'),
                      trailing: Column(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          Text('${item.retailPrice} دج', style: TextStyle(color: Colors.green, fontWeight: FontWeight.bold)),
                          Text('جملة: ${item.wholesalePrice}', style: TextStyle(fontSize: 10)),
                        ],
                      ),
                      onLongPress: () => _deleteProduct(index), // حذف عند الضغط المطول
                    ),
                  );
                },
              ),
        floatingActionButton: FloatingActionButton(
          onPressed: _addProduct,
          child: Icon(Icons.add),
          backgroundColor: Colors.teal,
        ),
      ),
    );
  }
}
flutter build apk --release

