## Step 1: Collect and Prepare Dataset
Download the dataset from Kaggle: Fruits Dataset.
Extract the dataset and organize it into folders based on different fruit or vegetable categories.

## Step 2: Train the Model using Teachable Machine
Visit Teachable Machine.
Click on "Get Started" and choose the "Image Project" option.
Upload your prepared dataset to teach the model.
Train the model using the provided options.
Export the model as a TensorFlow Lite (TFlite) model.

## Step 3: Create Flutter Application
Set up a new Flutter project using the following command:

```bash

flutter create fruit_classifier
cd fruit_classifier
```

Replace the contents of lib/main.dart with the following code:
```dart

import 'package:flutter/material.dart';
import 'package:tflite/tflite.dart';
import 'dart:io';
import 'package:image_picker/image_picker.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Fruit Classifier',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  File _image;
  List _output;

  @override
  void initState() {
    super.initState();
    loadModel();
  }

  Future<void> loadModel() async {
    await Tflite.loadModel(
      model: "assets/fruit_model.tflite",
      labels: "assets/fruit_labels.txt",
    );
  }

  Future<void> _classifyImage() async {
    var pickedImage = await ImagePicker().getImage(source: ImageSource.gallery);

    if (pickedImage == null) return;

    File imageFile = File(pickedImage.path);

    var output = await Tflite.runModelOnImage(
      path: imageFile.path,
      numResults: 5,
      threshold: 0.2,
    );

    setState(() {
      _image = imageFile;
      _output = output;
    });
  }

  @override
  void dispose() {
    Tflite.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Fruit Classifier'),
      ),
      body: Column(
        children: [
          _image == null ? Placeholder() : Image.file(_image),
          SizedBox(height: 20),
          ElevatedButton(
            onPressed: _classifyImage,
            child: Text('Classify Image'),
          ),
          _output != null
              ? Column(
                  children: _output.map((result) {
                    return Text('${result['label']} : ${result['confidence']}');
                  }).toList(),
                )
              : Container(),
        ],
      ),
    );
  }
}
```
Save the Teachable Machine exported TFlite model as assets/fruit_model.tflite and labels as assets/fruit_labels.txt.

## Step 4: Run the Application
Run the Flutter app using:

```bash

flutter run
```
