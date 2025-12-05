# CloudEats - Cloud-Native Food Delivery App 🍔

**CloudEats** is a real-time food delivery application simulation built to demonstrate cloud-native architecture using AWS. It features a live web frontend, a native mobile application, a containerized backend, and serverless automation to simulate kitchen operations.

## 🏗 Architecture

| Component | Technology Used | Role |
| :--- | :--- | :--- |
| **Backend API** | Node.js + Express | Handles order requests (Hosted on **AWS EKS**) |
| **Database** | AWS DynamoDB | Stores order details and status |
| **Web Frontend** | HTML/JS | Live website hosted on **AWS S3** |
| **Mobile App** | React Native (Expo) | Android/iOS app for placing orders |
| **Kitchen Logic** | AWS Lambda | Serverless trigger to simulate cooking & delivery updates |

---

## 🚀 Phase 1: Live Web Hosting (AWS S3)

Move the frontend from `localhost` to the public internet using AWS S3 Static Website Hosting.

### Deployment Steps:
1.  **Create S3 Bucket:**
    *   Name: `cloudeats-app-yourname` (Region: `us-east-1`).
    *   **Uncheck** "Block all public access".
2.  **Upload Frontend:**
    *   Rename your main HTML file to `index.html`.
    *   Upload to the bucket.
3.  **Configure Permissions:**
    *   Go to **Permissions** -> **Bucket Policy** and paste:
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
            }
        ]
    }
    ```
    *(Replace `YOUR_BUCKET_NAME` with your actual bucket name)*.

    ![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/d8227e88fd8f0f30d4786c94a1fdbc06fa2bae52/CloudEats/Screenshot%20(89).png)
    
5.  **Enable Hosting:**
    *   Go to **Properties** -> **Static website hosting**.
    *   Select **Enable**.
    *   Set Index document to `index.html`.
  
**Result:** Your app is now live at the provided S3 Endpoint URL.

 ![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/a528201925a383edf68ce9867a7b98d983970028/Screenshot%20(75).png)


---

## 📱 Phase 2: Mobile Application (React Native)

Build a real `.apk` capable mobile application that communicates with the AWS Backend.

### Setup
1.  Initialize the project:
    ```bash
    npx create-expo-app CloudEatsMobile
    cd CloudEatsMobile
    npx expo install axios
    ```

2.  **App Logic:** Replace `App.js` with the following code.
    *   *Note: Replace `YOUR_LOAD_BALANCER_URL` with your actual AWS EKS/Load Balancer endpoint.*

    ```javascript
    import React, { useState } from 'react';
    import { StyleSheet, Text, View, Image, TouchableOpacity, Alert, ActivityIndicator } from 'react-native';
    import axios from 'axios';

    export default function App() {
      const [loading, setLoading] = useState(false);
      
      // YOUR AWS BACKEND URL
      const API_URL = "http://YOUR_LOAD_BALANCER_URL/order";

      const placeOrder = async () => {
        setLoading(true);
        try {
          const payload = {
            item: "Chicken Biryani",
            restaurant: "Paradise",
            customer_name: "Mobile User"
          };

          const response = await axios.post(API_URL, payload);
          
          Alert.alert("Success! 🍔", `Order ID: ${response.data.order.orderId}`);
        } catch (error) {
          Alert.alert("Error", "Could not connect to Cloud Backend");
        } finally {
          setLoading(false);
        }
      };

      return (
        <View style={styles.container}>
          <Image 
            source={{ uri: 'https://source.unsplash.com/150x150/?burger' }} 
            style={styles.image} 
          />
          <Text style={styles.title}>CloudEats Mobile</Text>
          <Text style={styles.subtitle}>Order from your phone!</Text>

          <TouchableOpacity style={styles.button} onPress={placeOrder}>
            {loading ? <ActivityIndicator color="#fff" /> : <Text style={styles.btnText}>ORDER NOW</Text>}
          </TouchableOpacity>
        </View>
      );
    }

    const styles = StyleSheet.create({
      container: { flex: 1, backgroundColor: '#fff', alignItems: 'center', justifyContent: 'center' },
      image: { width: 150, height: 150, borderRadius: 20, marginBottom: 20 },
      title: { fontSize: 24, fontWeight: 'bold', color: '#333' },
      subtitle: { fontSize: 16, color: '#666', marginBottom: 30 },
      button: { backgroundColor: '#fc8019', padding: 15, borderRadius: 10, width: 200, alignItems: 'center' },
      btnText: { color: 'white', fontWeight: 'bold', fontSize: 18 }
    });
    ```

3.  **Run:**
    ```bash
    npx expo start
    ```
    Scan the QR code with the **Expo Go** app on your phone.

---

## 🤖 Phase 3: Automated "Kitchen" (AWS Lambda)

Serverless automation to update order statuses automatically using DynamoDB Streams.

1.  **Create Lambda Function:**
    *   Name: `KitchenWorker`
    *   Runtime: Node.js 18.x

2.  **Lambda Code:**
    ```javascript
    const AWS = require('aws-sdk');
    const dynamo = new AWS.DynamoDB.DocumentClient();

    exports.handler = async (event) => {
        // This runs whenever a new order is added to DynamoDB
        for (const record of event.Records) {
            if (record.eventName === 'INSERT') {
                const newOrder = AWS.DynamoDB.Converter.unmarshall(record.dynamodb.NewImage);
                
                // Wait 10 seconds (Simulate cooking)
                await new Promise(r => setTimeout(r, 10000));
                
                // Update Status to DELIVERED
                const params = {
                    TableName: "CloudEats-Orders",
                    Key: { orderId: newOrder.orderId },
                    UpdateExpression: "set #s = :status",
                    ExpressionAttributeNames: { "#s": "status" },
                    ExpressionAttributeValues: { ":status": "OUT_FOR_DELIVERY" }
                };
                
                await dynamo.update(params).promise();
                console.log(`Order ${newOrder.orderId} is now OUT FOR DELIVERY`);
            }
        }
    };
    ```

3.  **Setup Trigger:**
    *   Enable **DynamoDB Streams** on the `CloudEats-Orders` table (View type: New and old images).
    *   Add DynamoDB as a trigger to the Lambda function.
  
    ![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/bff86d911ff852d21547312b6eb060d01e09465c/Screenshot%20(90).png)

---
## 📸 Project Demo

Here is the application running live on both Web and Mobile platforms, connecting to the AWS Cloud.

| **Web Dashboard (S3 Hosted)** | **Mobile App (Real Device)** | **Live Order Processing** |
|:---:|:---:|:---:|
| <![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/e389b98b0cd5bb4e39218c02b3321482b7ac03c4/Screenshot%20(77).png)> | <![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/c3a3761779112534038f20099b91a1e010df7460/1764949168519.jpg)> | <![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/e916075bb2b70e790211ea95717c32f0a4ffcd6a/Screenshot%20(81).png)> |
| *Browsing the menu on the live website* | *Native Android App connected to AWS Load Balancer* | *Real-time UUID generation & DB confirmation* |

---

## 🏗 Architecture

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Backend API** | Node.js + Express | REST API handling orders (Hosted on **AWS EKS**) |
| **Database** | AWS DynamoDB | NoSQL database for high-speed order storage |
| **Web Frontend** | HTML5/CSS/JS | Static website hosted on **AWS S3** |
| **Mobile App** | React Native (Expo) | Android/iOS app for placing orders remotely |
| **Automation** | AWS Lambda | Serverless trigger simulating kitchen status updates |

---

## 🚀 Features implemented

*   ✅ **Cross-Platform Ordering:** Users can order from the web dashboard or their physical mobile device.
*   ✅ **Cloud Hosting:** The frontend is not on localhost; it is served globally via AWS S3.
*   ✅ **Container Orchestration:** The backend API runs on a Kubernetes Cluster (EKS) for scalability.
*   ✅ **Real-Time Data:** When an order is placed, a unique Order ID (UUID) is generated and stored in DynamoDB instantly.
*   ✅ **Automated Workflow:** AWS Lambda listens to database streams to automatically update order status from "PREPARING" to "OUT_FOR_DELIVERY".

---

## 🛠 Deployment & Setup

### Phase 1: The Web Frontend (AWS S3)
The web interface is hosted using AWS S3 Static Website Hosting.
1.  Upload `index.html` to an S3 bucket.
2.  Configure **Bucket Policy** for public read access.
3.  Enable **Static Website Hosting** in S3 properties.
4.  *Result:* A public URL accessible from any browser.

### Phase 2: The Mobile App (React Native)
Built using Expo to run on physical devices.
1.  **Install Dependencies:**
    ```bash
    npx create-expo-app CloudEatsMobile
    cd CloudEatsMobile
    npx expo install axios
    ```
2.  **Configure API:**
    Update `App.js` with the AWS Load Balancer URL.
3.  **Run:**
    ```bash
    npx expo start
    ```
    (Scan the QR code with the Expo Go app to test on a real phone).

### Phase 3: The Backend Logic (Lambda & DynamoDB)
Serverless functions handle the "Kitchen" logic.
*   **Trigger:** DynamoDB Stream (New Image).
*   **Action:** Lambda function waits 10 seconds (simulating cooking) and updates the status to `OUT_FOR_DELIVERY`.

---

## 📝 API Usage

The mobile app and website communicate with the backend via the following endpoint:

**POST** `/order`
```json
{
  "item": "Hyderabadi Chicken Biryani",
  "restaurant": "Paradise",
  "customer_name": "Cloud User"
}
Response:
code
JSON
{
  "message": "Order Placed Successfully",
  "order": {
    "orderId": "a33f713d-92a9-499e-a016-8a7a3cbc5f8e",
    "status": "PREPARING"
  }
```

### Phase 4: The "Kitchen" & Notifications (Lambda + SNS)
This phase automates the order flow and sends emails.

1.  **Create SNS Topic:**
    *   Create a Standard Topic named `CloudEats-Updates`.
    *   Create a Subscription (Protocol: Email) and confirm it in your inbox.
      
![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/01e36e56e5fb08ae945f2eb537dffe8c0e48d30d/Screenshot%20(86).png)

```
```
2.  **The Lambda Logic:**
    *   Trigger: DynamoDB Stream (New Image).
    *   Action: Waits 10s -> Updates DB -> Publishes to SNS.

![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/01e36e56e5fb08ae945f2eb537dffe8c0e48d30d/Screenshot%20(85).png)

```
```

![image alt](https://github.com/Naveenaece2000/CloudEats-Food-Delivery-App/blob/01e36e56e5fb08ae945f2eb537dffe8c0e48d30d/Screenshot%20(87).png)

```
```

**Lambda Code Snippet:**
```javascript
const AWS = require('aws-sdk');
const dynamo = new AWS.DynamoDB.DocumentClient();
const sns = new AWS.SNS();

exports.handler = async (event) => {
    for (const record of event.Records) {
        if (record.eventName === 'INSERT') {
            const newOrder = AWS.DynamoDB.Converter.unmarshall(record.dynamodb.NewImage);
            
            // 1. Simulate Cooking (Wait 10s)
            await new Promise(r => setTimeout(r, 10000));
            
            // 2. Update Status in DynamoDB
            const params = {
                TableName: "CloudEats-Orders",
                Key: { orderId: newOrder.orderId },
                UpdateExpression: "set #s = :status",
                ExpressionAttributeNames: { "#s": "status" },
                ExpressionAttributeValues: { ":status": "OUT_FOR_DELIVERY" }
            };
            await dynamo.update(params).promise();
            
            // 3. Send Email via SNS
            await sns.publish({
                TopicArn: "arn:aws:sns:us-east-1:YOUR_ACCOUNT_ID:CloudEats-Updates",
                Message: `Good news! Your order ${newOrder.orderId} is now OUT FOR DELIVERY! 🍔`,
                Subject: "CloudEats Order Update"
            }).promise();
            
            console.log("Order processed and email sent.");
        }
    }
};
}

