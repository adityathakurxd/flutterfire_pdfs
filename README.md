# FlutterFire PDFs
Working with PDFs in Flutter and Firebase.

## Firebase x Flutter
Setup Firebse for your project as you normally would. Steps [here](https://firebase.flutter.dev/docs/overview/).
Before any of the Firebase services can be used, FlutterFire needs to be initialized. We do this in `main.dart`
```dart
import 'package:flutter/material.dart';  
import 'screens/homepage.dart';  
import 'package:firebase_core/firebase_core.dart';  
  
void main() {  
  WidgetsFlutterBinding.ensureInitialized();  
  runApp(App());  
}  
  
class App extends StatelessWidget {  
  // Create the initialization Future outside of `build`:  
  final Future<FirebaseApp> _initialization = Firebase.initializeApp();  
  
  @override  
  Widget build(BuildContext context) {  
    return FutureBuilder(  
      // Initialize FlutterFire:  
  future: _initialization,  
      builder: (context, snapshot) {  
        // Check for errors  
  if (snapshot.hasError) {  
          return Text('Error in Firebase Initilisation');  
        }  
        // Once complete, show your application  
  if (snapshot.connectionState == ConnectionState.done) {  
          return MaterialApp(  
            title: 'College App',  
            home: HomePage(),  
          );  
        }  
        // Otherwise, show something whilst waiting for initialization to complete  
  return CircularProgressIndicator();  
      },  
    );  
  }  
}
```
## Working with PDFs

**Dependencies used:**
```dart
firebase_core: ^0.7.0   
firebase_storage: ^7.0.0  

flutter_document_picker: ^4.0.0  
flutter_plugin_pdf_viewer: ^1.0.7
```
**Note:**  I have used `flutter_document_picker: ^4.0.0 ` instead of `file_picker: ^2.1.5+1` because the latter was showing up an error related AndroidX incompatibilites of plugin which I'll dicuss in a YouTube video soon.

**Import the required classes to `homepage.dart` file:**
```dart
import 'dart:io';  
import 'dart:math';  
import 'package:flutter_document_picker/flutter_document_picker.dart';  
import 'package:firebase_storage/firebase_storage.dart' as firebase_storage;  
import 'package:flutter/material.dart';
```

**Create a Stateful Widget for HomePage**
```dart
class HomePage extends StatefulWidget {  
  @override  
  _HomePageState createState() => _HomePageState();  
}  
  
class _HomePageState extends State<HomePage> {  
  @override  
  Widget build(BuildContext context) {  
    return Container();  
  }  
}
```
We'll work in this widget to:

 1. Upload PDF from device storage to Firebase Storage
 2. Download  and View PDF in App


## Upload PDF from device storage to Firebase Storage
We'll first create a Scaffold with an App Bar and a Floating Action Button to open File Storage and select a PDF file.
Therefore, inside the HomePageState make the required changes.
```dart
class _HomePageState extends State<HomePage> {  
  @override  
  Widget build(BuildContext context) {  
    return Scaffold(  
      appBar: AppBar(  
        backgroundColor: Colors.blue,  
        title: Text("FlutterFire PDF"),  
      ),  
      floatingActionButton: FloatingActionButton(  
        backgroundColor: Colors.purple,  
        child: Icon(  
          Icons.add,  
          color: Colors.white,  
        ),  
        onPressed: () async {  
          //Functionality for Button to pick pdf will go here.
        },  
      ),  
    );  
  }  
}
```

Once done, we need to now add some functionality to the onPressed of the FloatingActionButton to pick a PDF from storage. We'll use the`flutter_document_picker: ^4.0.0 ` and make it async.

```dart
onPressed: () async {   
  final path = await FlutterDocumentPicker.openDocument();  
  print(path);  
  File file = File(path);  
  //firebase_storage.UploadTask task = await uploadFile(file);    
},
```
Now that we have the file, we need to upload it. Uncomment the uploadFile function in the onPressed so it'll look like this:
```dart
onPressed: () async {   
  final path = await FlutterDocumentPicker.openDocument();  
  print(path);  
  File file = File(path);  
  firebase_storage.UploadTask task = await uploadFile(file);    
},
```
**and, let us define this uploadFile function.**

```dart  
Future<firebase_storage.UploadTask> uploadFile(File file) async {  
  if (file == null) {  
    Scaffold.of(context)  
        .showSnackBar(SnackBar(content: Text("Unable to Upload")));  
    return null;  
  } 

  firebase_storage.UploadTask uploadTask;  
  
  // Create a Reference to the file  
  firebase_storage.Reference ref = firebase_storage.FirebaseStorage.instance  
  .ref()  
      .child('playground')  
      .child('/some-file.pdf');  
  
  final metadata = firebase_storage.SettableMetadata(  
      contentType: 'file/pdf',  
      customMetadata: {'picked-file-path': file.path});  
  print("Uploading..!");  
  
  uploadTask = ref.putData(await file.readAsBytes(), metadata);  
  
  print("done..!");  
  return Future.value(uploadTask);  
}
```
The file would be saved inside playground as some-file.pdf on your Cloud Storage.

## View PDF from Firebase Storage

To work with PDF Viewer we'll use two classes: a `loader.dart` and a `viewer.dart`. The homepage widget is where we get URL from Firebase Storage and then display PDF using URL in viewer widget.

**Imports used:**
```dart
import 'package:college_app/screens/notes.dart';  
import 'package:flutter/material.dart';  
import 'package:firebase_storage/firebase_storage.dart' as firebase_storage;  
import 'package:flutter_plugin_pdf_viewer/flutter_plugin_pdf_viewer.dart';
```
 **Initialise a LoadURL Stateful Widget:**
 ```dart
 class LoadURL extends StatefulWidget {  
  @override  
  _LoadURLState createState() => _LoadURLState();  
}  
  
class _LoadURLState extends State<LoadURL> {  
  @override  
  Widget build(BuildContext context) {  
    return Container();  
  }  
}
```

**Initialise an instance of Firebase Storage:**
```dart
firebase_storage.FirebaseStorage storage = firebase_storage.FirebaseStorage.instance;
```
**Define a function to list all the directories on Storage:**
```dart
Future<void> listExample() async {  
  firebase_storage.ListResult result =  
  await firebase_storage.FirebaseStorage.instance.ref().child('notes').listAll();  
  
  result.items.forEach((firebase_storage.Reference ref) {  
    print('Found file: $ref');  
  });  
  
  result.prefixes.forEach((firebase_storage.Reference ref) {  
    print('Found directory: $ref');  
  });  
}
```
**Define a function to download URL:**
```dart
Future<void> downloadURLExample() async {  
  String downloadURL = await firebase_storage.FirebaseStorage.instance  
  .ref('notes/name.pdf')  
      .getDownloadURL();  
  print(downloadURL);  
  PDFDocument doc = await PDFDocument.fromURL(downloadURL);  
  Navigator.push(context, MaterialPageRoute(builder: (context)=>ViewPDF(doc)));  //Notice the Push Route once this is done.
}
```
**Note**: You can use the directories found from listExample function and save them to a variable and use string interpolation to define path in downloadURLExample function.

**Inside the `initState` we call the two functions:**
```dart
@override  
void initState() {  
  // TODO: implement initState  
  super.initState();    
  listExample();  
  downloadURLExample();  
  print("All done!");  
}
```

While the `initState` works in the background to fetch files, the builder can return a `CircularProgressIndicator()`:
```dart
@override  
Widget build(BuildContext context) {  
  return Center(  
    child: CircularProgressIndicator(  
    ),  
  );  
}
```
**When the `initState` is done, a new widget will be pushed in from `viewer.dart`.**

**Initialise a Stateful Widget:**
```dart
class ViewPDF extends StatefulWidget {  
  @override  
  _ViewPDFState createState() => _ViewPDFState();  
}  
  
class _ViewPDFState extends State<ViewPDF> {  
  @override  
  Widget build(BuildContext context) {  
    return Container();  
  }  
}
```

**Create a constructor that'll take in a `PDFDocument document` as parameter and display the pdf in builder.**
```dart
class ViewPDF extends StatefulWidget {  
  PDFDocument document;  
  ViewPDF(this.document);  
  @override  
  _ViewPDFState createState() => _ViewPDFState();  
}  
  
class _ViewPDFState extends State<ViewPDF> {  
  @override  
  Widget build(BuildContext context) {  
    return Center(  
        child: PDFViewer(document: widget.document));  
  }  
}
```
and, this should load up the PDF when run :)
