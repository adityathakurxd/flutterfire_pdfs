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

