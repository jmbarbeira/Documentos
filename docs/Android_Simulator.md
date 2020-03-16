---
id: Android_Simulator
title: Getting started with the simulator
---

**Getting started with the simulator**

Introduction

This tutorial is guiding you through all the steps to create a basic payment application for Android devices using a card reader simulator. The simulator only has limited capabilities and we highly recommend that you order a development kit if you want to carry a full integration. The development kit contains a card reader as well as a test card and will allow you to test your integration from end to end.

The new generation of Handpoint SDK's is designed to make your life easier. Simple and created for humans, it does not require any specific knowledge of the payment industry to be able to start accepting credit/debit card transactions.

At Handpoint we take care of securing every transaction so you don´t have to worry about it while creating your application. We encrypt data from the payment terminal to the bank with our point-to-point encryption solution. Our platform is always up to the latest PCI-DSS security requirements. We undergo an audit on a yearly basis to be able to maintain the license to handle, process, store and transmit card data.

Connecting to the simulator

The SDK offers a method in which you will need to specify the card reader to be used:

```C
hapi.useDevice(new Device("Name", "Port", "Address", ConnectionMethod.****))
```
Simply set the ConnectionMethod to Simulator, i.e. ConnectionMethod.Simulator. The SDK does the rest. You don't need to search via bluetooth for surrounding card readers when using the simulator.

```C
hapi.useDevice(new Device("Name", "Port", "Address", ConnectionMethod.Simulator))
```
Controlling responses

The simulator mimics the card reader as much as possible regarding information flow from the SDK interface to your application. It will return all the transaction statuses, transaction results and receipts.

Results of a transaction are controlled by the amount sent into the sale function:
The 3rd position from the right sets the desired financial status, 0 = Authorized and 1 = Declined.
The 4th position from the right sets the desired verification method, 0 = Signature and 1 = PIN.
```
hapi.Sale(X10XX, Currency.GBP); // amount = X 10 XX - where X represents an integer [0;9]
``
X 00 XX = Signature authorized
X 01 XX = Signature declined
X 10 XX = Pin authorized
X 11 XX = Pin declined

Let's start programming!

Modify the AndroidManifest.xml
The following lines have to be added inside the manifest in order to enable permissions:

```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.INTERNET"/>
``
In order for the application to handle screen rotation the following line of code has to be added inside your manifest under <activity>:

```
<activity android:configChanges="orientation|keyboardHidden|screenSize">
``
According to the android version of the device you are using you will have to modify the following line of your manifest:

```
<uses-sdk android:minSdkVersion="14"/>//change the SDK version to the one corresponding to the device you are using
```
Your manifest should look like this:

``
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.example.androidGettingStarted"
            android:versionCode="1"
            android:versionName="1.0">
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-sdk android:minSdkVersion="14"/>
    <application android:label="@string/app_name" android:icon="@drawable/ic_launcher">
        <activity android:name="MyActivity"
                android:configChanges="orientation|keyboardHidden|screenSize"
                android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```
Create a Java class
Create a new java class called MyClass.java and include com.handpoint.api.* as a dependency:

```
package com.example.androidGettingStarted;

import com.handpoint.api.*;

public class MyClass {
}

Initialize the API

package com.example.androidGettingStarted;

import android.content.Context;
import com.handpoint.api.*;

public class MyClass  {

    Hapi api;

    public MyClass(Context context) {
        initApi(context);
    }

    public void initApi(Context context) {
        String sharedSecret = "0102030405060708091011121314151617181920212223242526272829303132";
        this.api = HapiFactory.getAsyncInterface(this, context).defaultSharedSecret(sharedSecret);
        // The api is now initialized. Yay! we've even set a default shared secret!
        // The shared secret is a unique string shared between the card reader and your mobile application.
        // It prevents other people to connect to your card reader.
    }
}

Implement the mandatory Events(Events.Required)

package com.example.androidGettingStarted;

import android.content.Context;
import com.handpoint.api.*;
import java.util.List;

public class MyClass implements Events.Required {

    Hapi api;

    public MyClass(Context context) {
        initApi(context);
    }
    public void initApi(Context context) {
        String sharedSecret = "0102030405060708091011121314151617181920212223242526272829303132";
        this.api = HapiFactory.getAsyncInterface(this, context).defaultSharedSecret(sharedSecret);
        // The api is now initialized. Yay! we've even set a default shared secret!
        // The shared secret is a unique string shared between the card reader and your mobile application.
        // It prevents other people to connect to your card reader.
    }

    @Override
    public void deviceDiscoveryFinished(List<Device> list) {
        // Only needed when using a card reader
        // Here you get a list of Bluetooth devices discovered around your android device
    }

    @Override
    public void signatureRequired(SignatureRequest signatureRequest, Device device) {
        // You'll be notified here if a sale process needs a signature verification
        // A signature verification is needed if the cardholder uses an MSR card or a chip & signature card
        // This method will not be invoked if a transaction is made with a Chip & PIN card
        // At this step, you are supposed to display the merchant receipt to the cardholder on the android device
        // the cardholder must have the possibility to accept or decline the transaction
        // If the cardholder clicks on decline, the transaction is VOID
        // If the cardholder clicks on accept he is then asked to sign electronically the receipt
        this.api.signatureResult(true);// This line means that the cardholder ALWAYS accepts to sign the receipt
        // For this sample app we are not going to implement the whole signature process
    }

    @Override
    public void endOfTransaction(TransactionResult transactionResult, Device device) {
        // The object TransactionResult stores a lot of information
        // For example, it holds the merchant receipt as well as the cardholder receipt
        // Other information can be accessed through this object like the transaction ID, the amount...
    }

}
```
Add a method to connect to the simulator
``
public void connect() {
    Device device = new Device("name", "address", "port", ConnectionMethod.SIMULATOR);
    this.api.useDevice(device);
}
```
Add a method to take payments with the simulator
```
hapi.Sale(X10XX, Currency.GBP); // amount = X 10 XX - where X represents an integer [0;9]
``
Let´s add 4 different functions to MyClass.java in order to represent the 4 main payment scenarios:

```
public boolean payWithSignatureAuthorized() {
    return this.api.sale(new BigInteger("10000"), Currency.GBP);
    // amount X00XX where X represents an integer [0;9] --> Signature authorized
}
public boolean payWithSignatureDeclined() {
    return this.api.sale(new BigInteger("10100"), Currency.GBP);
    // amount X01XX where X represents an integer [0;9] --> Signature declined
}
public boolean payWithPinAuthorized() {
    return this.api.sale(new BigInteger("11000"), Currency.GBP);
    // amount X10XX where X represents an integer [0;9] --> PIN authorized
}
public boolean payWithPinDeclined() {
    return this.api.sale(new BigInteger("11100"), Currency.GBP);
    // amount X11XX where X represents an integer [0;9] --> PIN declined
}
``
Let's create a User Interface!

Create buttons
Go to the layout folder of your project and add the following code to the xml file containing the code for your user interface:


<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:orientation="vertical"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent">
<Button
            android:id="@+id/connectSimulator"
            android:text="Connect To Simulator"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
<Button
            android:id="@+id/payWithSignatureAuthorized"
            android:text="Pay With Signature Authorized"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
<Button
            android:id="@+id/payWithSignatureDeclined"
            android:text="Pay With Signature Declined"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
<Button
            android:id="@+id/payWithPinAuthorized"
            android:text="Pay With Pin Authorized"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />
<Button
            android:id="@+id/payWithPinDeclined"
            android:text="Pay With PIN Declined"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content" />

</LinearLayout>

Initialize the buttons
Go to your main activity (entry point of your program) and create new methods to initialize the buttons:


package com.example.androidGettingStarted;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;


public class MyActivity extends Activity {
    private Button connectToSimulatorButton;
    private Button payWithSignatureAuthorizedButton;
    private Button payWithSignatureDeclinedButton;
    private Button payWithPinAuthorizedButton;
    private Button payWithPinDeclinedButton;
    /**
    * Called when the activity is first created.
    */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        initializeButtons();
    }
    public void initializeButtons() {
    connectToSimulatorButton = (Button) findViewById(R.id.connectSimulator);
    connectToSimulatorButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        //Call an OnClick method
        }
    });

    payWithSignatureAuthorizedButton = (Button) findViewById(R.id.payWithSignatureAuthorized);
    payWithSignatureAuthorizedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        //Call an OnClick method
        }
    });

    payWithSignatureDeclinedButton = (Button) findViewById(R.id.payWithSignatureDeclined);
    payWithSignatureDeclinedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        //Call an OnClick method
        }
    });

    payWithPinAuthorizedButton = (Button) findViewById(R.id.payWithPinAuthorized);
    payWithPinAuthorizedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        //Call an OnClick method
        }
    });

    payWithPinDeclinedButton = (Button) findViewById(R.id.payWithPinDeclined);
    payWithPinDeclinedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        //Call an OnClick method
        }
    });
}

Let's link our user interface with methods!

Referencing your main activity in MyClass
To be able to call methods from your main activity in MyClass you need to define your main activity as a Context in MyClass and then define a new instance of it:


public class MyClass implements Events.Required {
    Hapi api;
    MyActivity UIClass;

public MyClass(MyActivity activity){
    initApi((Context) activity);
    UIClass = activity;
}

Referencing Myclass in your main activity
To be able to call methods from MyClass to your main activity you need to instantiate MyClass in your main activity:


public class MyActivity extends Activity{
    private Button connectToSimulatorButton;
    private Button payWithSignatureAuthorizedButton;
    private Button payWithSignatureDeclinedButton;
    private Button payWithPinAuthorizedButton;
    private Button payWithPinDeclinedButton;
    MyClass myClass;

    /**
    * Called when the activity is first created.
    */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        myClass = new MyClass(this);
        initializeButtons();
    }

Add OnClick methods when using the buttons
Let's call specific methods when clicking on the different buttons:


public void initializeButtons() {
    connectToSimulatorButton = (Button) findViewById(R.id.connectSimulator);
    connectToSimulatorButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        myClass.connect();
        }
    });
    payWithSignatureAuthorizedButton = (Button) findViewById(R.id.payWithSignatureAuthorized);
    payWithSignatureAuthorizedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        myClass.payWithSignatureAuthorized();
        }
    });
    payWithSignatureDeclinedButton = (Button) findViewById(R.id.payWithSignatureDeclined);
    payWithSignatureDeclinedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        myClass.payWithSignatureDeclined();
        }
    });
    payWithPinAuthorizedButton = (Button) findViewById(R.id.payWithPinAuthorized);
    payWithPinAuthorizedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        myClass.payWithPinAuthorized();
        }
    });
    payWithPinDeclinedButton = (Button) findViewById(R.id.payWithPinDeclined);
    payWithPinDeclinedButton.setOnClickListener(new View.OnClickListener() {
        public void onClick(View view) {
        myClass.payWithPinDeclined();
        }
    });
}

Let's implement the android lifecycle methods!

Implementing your activity lifecycle methods properly ensures your app behaves well in several ways, including that it:

Does not crash if the user receives a phone call or switches to another app while using your app
Does not consume valuable system resources when the user is not actively using it
Does not lose the user's progress if they leave your app and return to it at a later time
Does not crash or lose the user's progress when the screen rotates between landscape and portrait orientation
Add the lifecycle methods in your main activity

@Override
public void onPause() {
    super.onPause();  // Always call the superclass method first
    myClass.disconnect(); // disconnects from the card reader if the application is paused
}

@Override
public void onResume() {
    super.onResume();  // Always call the superclass method first
}

@Override
public void onBackPressed() {
    moveTaskToBack(true); // When the user clicks the back button the application is moved in the background
}

@Override
public void onStop() {
    super.onStop();  // Always call the superclass method first
}

@Override
public void onRestart() {
    super.onRestart();  // Always call the superclass method first
    initializeButtons(); // initialize the buttons again after restarting the app
}

@Override
public void onDestroy() {
    super.onDestroy();
}

Let´s display a receipt at the end of a transaction!

in this example, you are going to display the customer receipt. However, to have your app accredited by Handpoint you need to display BOTH the customer receipt AND the merchant receipt at the end of a transaction. We decided to only display one receipt to simplify this basic application as much as possible.

Fetch the cardholder's receipt from the method EndOfTransaction in MyClass

@Override
public void endOfTransaction(TransactionResult transactionResult, Device device) {
    // The object TransactionResult stores a lot of information, for example, it holds the different receipts
    // Other information can be accessed through this object like the transaction ID, the currency, the amount...
    UIClass.callReceiptDialog(transactionResult.getCustomerReceipt());
}

Create an AlertDialog in your main activity to display the receipt on the android device

public void callReceiptDialog(final String customerReceipt) {
    this.runOnUiThread(new Runnable() {
        @Override
        public void run() {
            String positiveButton = "OK";
            WebView webView = new WebView(MyActivity.this); // Create a webView to display the receipt
            webView.loadData(customerReceipt, "text/html", "UTF-8");
            webView.getSettings().setDefaultFontSize(20);
            webView.setVerticalScrollBarEnabled(true);
            new AlertDialog.Builder(MyActivity.this)// Defines an AlertDialog that will popup
                .setTitle("Transaction result")
                .setPositiveButton(positiveButton, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        dialogInterface.dismiss();
                    }
                })
                .setView(webView)
                .show();
        }
    });
}

Final Result!

Here is how MyClass and your main activity must eventually look like:

main activity (entry point for your program):

package com.example.androidGettingStarted;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.DialogInterface;
import android.os.Bundle;
import android.view.View;
import android.webkit.WebView;
import android.widget.Button;
import com.handpoint.api.TransactionResult;


public class MyActivity extends Activity{
    private Button connectToSimulatorButton;
    private Button payWithSignatureAuthorizedButton;
    private Button payWithSignatureDeclinedButton;
    private Button payWithPinAuthorizedButton;
    private Button payWithPinDeclinedButton;
    MyClass myClass;

    /**
    * Called when the activity is first created.
    */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        myClass = new MyClass(this);
        initializeButtons();
    }
    public void initializeButtons() {
        connectToSimulatorButton = (Button) findViewById(R.id.connectSimulator);
        connectToSimulatorButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
            myClass.connect();
            }
        });
        payWithSignatureAuthorizedButton = (Button) findViewById(R.id.payWithSignatureAuthorized);
        payWithSignatureAuthorizedButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
            myClass.payWithSignatureAuthorized();
            }
        });
        payWithSignatureDeclinedButton = (Button) findViewById(R.id.payWithSignatureDeclined);
        payWithSignatureDeclinedButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
            myClass.payWithSignatureDeclined();
            }
        });
        payWithPinAuthorizedButton = (Button) findViewById(R.id.payWithPinAuthorized);
        payWithPinAuthorizedButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
            myClass.payWithPinAuthorized();
            }
        });
        payWithPinDeclinedButton = (Button) findViewById(R.id.payWithPinDeclined);
        payWithPinDeclinedButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
            myClass.payWithPinDeclined();
            }
        });
    }
    public void callReceiptDialog(final String customerReceipt) {
        this.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                String positiveButton = "OK";
                WebView webView = new WebView(MyActivity.this); // Create a webView to display the receipt
                webView.loadData(customerReceipt, "text/html", "UTF-8");
                webView.getSettings().setDefaultFontSize(20);
                webView.setVerticalScrollBarEnabled(true);
                new AlertDialog.Builder(MyActivity.this)// Defines an AlertDialog that will popup
                    .setTitle("Transaction result")
                    .setPositiveButton(positiveButton, new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialogInterface, int i) {
                            dialogInterface.dismiss();
                        }
                    })
                    .setView(webView)
                    .show();
            }
        });
    }

    @Override
    public void onPause() {
        super.onPause();  // Always call the superclass method first
        myClass.disconnect(); // disconnects from the card reader if the application is paused
    }

    @Override
    public void onResume() {
        super.onResume();  // Always call the superclass method first
    }

    @Override
    public void onBackPressed() {
        moveTaskToBack(true); // the application is moved to the background
    }

    @Override
    public void onStop() {
        super.onStop();  // Always call the superclass method first
    }

    @Override
    public void onRestart() {
        super.onRestart();  // Always call the superclass method first
        initializeButtons(); // initialize the buttons again after restarting the app
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}

My Class :

package com.example.androidGettingStarted;
import java.math.BigInteger;
import android.content.Context;
import com.handpoint.api.*;

import java.util.List;

public class MyClass implements Events.Required {
    Hapi api;
    MyActivity UIClass;

    public MyClass(MyActivity activity){
        initApi((Context) activity);
        UIClass = activity;
    }
    public void initApi(Context context) {
        String sharedSecret = "0102030405060708091011121314151617181920212223242526272829303132";
        this.api = HapiFactory.getAsyncInterface(this, context).defaultSharedSecret(sharedSecret);
        // The api is now initialized. Yay! we've even set a default shared secret!
        // The shared secret is a unique string shared between the card reader and your mobile application.
        // It prevents other people to connect to your card reader.
    }

    @Override
    public void deviceDiscoveryFinished(List<Device> list) {
        // Only needed when using a payment terminal, here you get a list of Bluetooth devices discovered
    }

    public void connect() {
        Device device = new Device("name", "address", "port", ConnectionMethod.SIMULATOR);
        this.api.useDevice(device);
    }


    public boolean payWithSignatureAuthorized() {
        return this.api.sale(new BigInteger("10000"), Currency.GBP);
        // amount X00XX where X represents an integer [0;9] --> Signature authorized
    }

    public boolean payWithSignatureDeclined() {
        return this.api.sale(new BigInteger("10100"), Currency.GBP);
        // amount X01XX where X represents an integer [0;9] --> Signature declined
    }

    public boolean payWithPinAuthorized() {
        return this.api.sale(new BigInteger("11000"), Currency.GBP);
        // amount X10XX where X represents an integer [0;9] --> PIN authorized
    }

    public boolean payWithPinDeclined() {
        return this.api.sale(new BigInteger("11100"), Currency.GBP);
        // amount X11XX where X represents an integer [0;9] --> PIN declined
    }

    @Override
    public void signatureRequired(SignatureRequest signatureRequest, Device device) {
        // You'll be notified here if a sale process needs a signature verification
        // A signature verification is needed if the cardholder uses an MSR card or a chip & signature card
        // This method will not be invoked if a transaction is made with a Chip & PIN card
        // At this step, you should display the merchant receipt to the cardholder on the android device
        // The cardholder must have the possibility to accept or decline the transaction
        // If the cardholder clicks on decline, the transaction is VOID
        // If the cardholder clicks on accept he is then asked to sign electronically the receipt
        this.api.signatureResult(true);
        // This line means that the cardholder ALWAYS accepts to sign the receipt
        // For this sample app we are not going to implement the whole signature process
    }

    @Override
    public void endOfTransaction(TransactionResult transactionResult, Device device) {
        // The object TransactionResult stores holds the different receipts
        // Other information can be accessed through this object like the transaction ID, the amount...
        UIClass.callReceiptDialog(transactionResult.getCustomerReceipt());
    }

}

Let's run our program!

Run the program by clicking the "play" button, ensure that your android device is connected via USB and correctly recognized by your computer. If not, you can download specific drivers on your device manufacturer website and install them.

On your android device, click on the "Connect to simulator" button and then click on one of the payment types available and have a look at the receipts! Voila!