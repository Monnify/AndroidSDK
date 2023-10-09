# AndroidSDK

### An Integration Guide for the Monnify Android SDK

With the Monnify Android SDK, you are able to receive payments in your Android application via:

*  Card
* Bank Transfer
* USSD
* Phone Number

Outline below are the necessary steps required to integrate the Android SDK

1. ##### Add the needed dependency for the Monnify SDK
  The Monnify SDK requires the google and mavenCentral libs to work, so in your **root build.gradle file**, add the required libs

   ```java
     allprojects {
       repositories {
           google()
           mavenCentral() 
           }
       }
   ```

   Then in your **app-level build.gradle file**, add the monnify sdk

   ```java
     dependencies {
       // ...
       implementation "com.monnify.android-sdk:monnify-android-sdk:1.1.7"
     }
   ```   
  

2. ##### Create an instance of the Monnify SDK
   Next you are to create a class and then instantiate the Monnify sdk as shown below:

   ```kotlin
   For Kotlin  
   class MainActivity : AppCompatActivity() {
    // ...

    var monnify = Monnify.instance

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        }

    }
   ```  

   ```java
   class MainActivity extends AppCompatActivity {
    // ...

    private Monnify monnify = Monnify.Companion.getInstance();

    @Override
	public void onCreate(Bundle savedInstanceState) {
        // ...
        }
    }
   ```  
     
3. ##### Setting your API credentials
   You can then set your Monnify API key and Contract code on the OnCreate method of the Monnify instance.
   Also, you can set the Monnify environment to be used on this method  

   ```kotlin
   Kotlin Implementation

   monnify.setApiKey("MY_MERCHANT_API_KEY7")
   monnify.setContractCode("111222333444555")

   monnify.setApplicationMode(ApplicationMode.TEST)
   ```

   ```java
   Java Implementation

   monnify.setApiKey("MY_MERCHANT_API_KEY7");
   monnify.setContractCode("111222333444555");

   monnify.setApplicationMode(ApplicationMode.TEST);
   ```

4. ##### Set the request code and key
   On clicking the 'Pay button', the initializePayment() method requires the activity context, an object of the TransactionDetails class, request code and result key. You can set the request code and key yourself in order to have more control, and to prevent clashes with other args in the main app.  
   ```kotlin
   Kotlin Implementation

   val transaction = TransactionDetails.Builder()
            .amount(BigDecimal("2000"))
            .currencyCode("NGN")
            .customerName("Customer Name")
            .customerEmail("mail.cus@tome.er")
            .paymentReference("PAYMENT_REF")
            .paymentDescription("Description of payment")
            .build()
            
   monnify.initializePayment(
               this@MainActivity,
               transaction,
               INITIATE_PAYMENT_REQUEST_CODE,
               KEY_RESULT)
   ```

   ```java
   Java Implementation

   TransactionDetails transaction = new TransactionDetails.Builder()
            .amount(new BigDecimal("2000"))
            .currencyCode("NGN")
            .customerName("Customer Name")
            .customerEmail("mail.cus@tome.er")
            .paymentReference("PAYMENT_REF")
            .paymentDescription("Description of payment")
            .build();

   monnify.initializePayment(
           MainActivity.this,
                   transaction,
                   INITIATE_PAYMENT_REQUEST_CODE,
                   KEY_RESULT);
   ```

5. ##### Update application UI after transaction completion
   Finally, you should listen for payment attempt outcomes on the OnActivityResult() method when the Monnify payment gateway closes. Use the request code and data key passed in the initializePayment() method to get response returned by the SDK.
   ```kotlin
   Kotlin Implementation

   override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        
        val monnifyTransactionResponse = data?.getParcelableExtra(KEY_RESULT) as MonnifyTransactionResponse

        var message = ""
        message = when(monnifyTransactionResponse.status) {
            Status.PENDING -> "Transaction not paid"
            Status.PAID -> "Customer paid exact amount"
            Status.OVERPAID -> "Customer paid more than expected amount."
            Status.PARTIALLY_PAID -> "Customer paid less than expected amount."
            Status.FAILED -> "Transaction completed unsuccessfully. This means no payment came in for Account Transfer method or attempt to charge card failed."
            Status.PAYMENT_GATEWAY_ERROR -> "Payment gateway error"
        }

        Toast.makeText(this@MainActivity, message, Toast.LENGTH_LONG).show()

    }
   ```

```java
   Java Implementation

   @Override
   public void onActivityResult(int requestCode, int resultCode, Intent data) {
       super.onActivityResult(requestCode, resultCode, data);
    
    MonnifyTransactionResponse monnifyTransactionResponse = (MonnifyTransactionResponse) data.getParcelableExtra(KEY_RESULT);
    
    if (monnifyTransactionResponse == null)
        return;

    String message = "";
    switch (monnifyTransactionResponse.getStatus()) {
        case PENDING: { message = "Transaction not paid for."; break; }
        case PAID: { message = "Customer paid exact amount"; break; }
        case OVERPAID: { message = "Customer paid more than expected amount."; break; }
        case PARTIALLY_PAID: { message = "Customer paid less than expected amount."; break; }
        case FAILED: { message = "Transaction completed unsuccessfully. This means no payment came in for Account Transfer method or attempt to charge card failed."; break; }
        case PAYMENT_GATEWAY_ERROR: { message = "Payment gateway error"; break; }
    }

    Toast.makeText(MainActivity.this, message, Toast.LENGTH_LONG).show();
}
```


#### Split Payments And MetaData
On transaction initialization, you might want certain percent or amount of incoming payments to be settled to different bank accounts other than the your default settlement destination on Monnify. Here's where **Sub-Accounts** come in handy. 
Furthermore, you might also want to receive other user or application related data different from the request parameters. You can capture such data using the metaData object parameter.
```kotlin
Kotlin Implementation

transaction = TransactionDetails.Builder()
            //...
            .incomeSplitConfig(arrayListOf<SubAccountDetails>(
                SubAccountDetails("MFY_SUB_319452883968", 10.5f, BigDecimal("500"), true),
                SubAccountDetails("MFY_SUB_259811283666", 10.5f, BigDecimal("1000"), false)
            ))
            .metaData(hashMapOf(
                Pair("deviceType", "mobile_android"),
                Pair("ip", "127.168.22.98")
                // any other info
            ))
            .paymentMethods(arrayListOf<PaymentMethod>(
                add(PaymentMethod.CARD),
                add(PaymentMethod.ACCOUNT_TRANSFER)
            ))
            .build() 
```

```java
Java Implementation

TransactionDetails transaction = new TransactionDetails.Builder()
            //...
            .incomeSplitConfig(new ArrayList<SubAccountDetails>() {{ 
                add(new SubAccountDetails("MFY_SUB_319452883968", 10.5f, new BigDecimal("500"), true)); 
                add(new SubAccountDetails("MFY_SUB_259811283666", 10.5f, new BigDecimal("1000"), false)); 
            }})
            .metaData(new HashMap<String, String>() {{
                put("deviceType", "mobile_android");
                put("ip", "127.168.22.98");
                // any other info
            }})
            .paymentMethods(new ArrayList<PaymentMethod>() {{
                add(PaymentMethod.CARD);
                add(PaymentMethod.ACCOUNT_TRANSFER);
            }})
            .build();

```

___
For further information, you can check out our [developer documentation](https://developers.monnify.com) and you can also join our [slack community](https://slack.monnify.com).


