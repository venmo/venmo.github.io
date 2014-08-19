---
layout: post
title:  "Building a Sample App with the Android App Switch SDK"
author: Matthew Gotteiner
---

The Venmo Android App Switch SDK enables anyone to easily add person-to-person payments to an app. After adding the required sample files to your project, all that's left is to add a few input fields and fire off a request. To demonstrate how simple it is to integrate the App Switch SDK, let's walk through a sample app that allows a user to send a Venmo payment to a contact by searching for a name or phone number. Integrating contacts into your app is not a requirement to use our SDK, but it is a common use case so I decided to go over that process here. Download the source code for the complete app here: https://github.com/mgottein/app-switch-demo

This tutorial uses an `AutoCompleteTextView` widget to search a user's contact list. `AutoCompleteTextView` is similar to `EditText`, except that it provides a list of suggestions to the user based on what they are typing and allows the user to click on one to complete their entry.

In order for `AutocompleteTextView` to know what suggestions to provide to the user, we must provide it a `ListAdapter` that implements `Filterable`. The `ListAdapter` allows the `AutocompleteTextView` to access your suggestions, and implementing `Filterable` filters the list by what the user is currently typing. We'll be using the `SimpleCursorAdapter ListAdapter`.

First, retrieve a user's contact information by adding the read contacts permission to  AndroidManifest.xml. If we don't add this permission, we will be prevented from reading contact info. Add this line under the manifest tag:

```XML
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

Contact information in Android is exposed by a `ContentProvider`, a class that allows applications to access content. The contact `ContentProvider` allows us to look up the contact information via a SQLite query. The contact information will be inside a `Cursor`, a class that encapsulates the query results. Since our data is inside a `Cursor`, we don't have to implement our own `ListAdapter` from scratch and can instead use a `SimpleCursorAdapter` which handles much of the adapter boilerplate code for us. 
We can access the contact `ContentProvider` by using a `ContentResolver`. `ContentResolver` provides a query method that looks like a SQLite query. Let's take a look at its definition:

```Java
ContentResolver.query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
```

- `uri` allows the `ContentResolver` to identify a `ContentProvider` that will handle this query. We will be using this parameter to address our query to the contacts `ContentProvider`
- `projection` is a list of columns we want the query to return
- `selection` is a list of predicates that allow us to filter out unwanted results
- `selectionArgs` is a list of arguments to our `WHERE` clause. Question marks will be replaced with the selection args in the order they appear (first question mark maps to `selectionArgs[0]`, …)
- `sortOrder` specifies how the results should be ordered in the cursor

This generates a SQLite query that will look something like this:

```SQL
SELECT <projection> FROM <uri> WHERE <selection> ORDER BY <sortOrder>
```


Making the Projection Statement
-------------------------------

There are many columns in the Android `Contact` entity, but we only need a few of them here. We care about `Contacts.DISPLAY_NAME_PRIMARY`, `Phone.NUMBER`, and `Data._ID`.

We can now set up our projection:

```Java
private static final String[] PROJECTION = new String[]{
    Data._ID,
    Contacts.DISPLAY_NAME_PRIMARY,
    Phone.NUMBER
};
```


Making the SQLite Selection Statement
-------------------------------------

Building SQLite queries in Java can be confusing, so let's walk through this step-by-step. 
- Check if either `Contact.DISPLAY_NAME_PRIMARY` or `Phone.NUMBER` start with the input text. Since the contact name and phone number are strings, we can use the `LIKE` operator to see if the input text matches. 

```Java
private final String SELECTION = '(' + Contacts.DISPLAY_NAME_PRIMARY + " LIKE ? OR " + Phone.NUMBER + " LIKE ?) AND " +
```
- Compare the `Phone.TYPE` column to the `Phone.TYPE_MOBILE` column to make sure we are only looking at mobile phone numbers.

```Java
Phone.TYPE + "='" + Phone.TYPE_MOBILE + "' AND " +
```

- Because there are many different types of information associated with a contact, compare the `Data.MIME_TYPE` column to `Phone.CONTENT_ITEM_TYPE` using the equality operator to make sure we are dealing with phone number information.

```Java
Data.MIMETYPE + "='" + Phone.CONTENT_ITEM_TYPE + "'";
```


Instantiating the Adapter
-------------------------

In order to instantiate the adapter, we need to provide a valid `Context`, a layout resource for a single list item, and a mapping between columns and view IDs. This mapping allows the `SimpleCursorAdapter` to automatically display a list item for us with very little code. We can choose to provide a cursor at instantiation, but for our purposes this is unnecessary. We are using a pre-existing layout resource for simplicity.

```Java
SimpleCursorAdapter adapter = new SimpleCursorAdapter(this,
    android.R.layout.simple_list_item_2,
    null,
    new String[]{
        Contacts.DISPLAY_NAME_PRIMARY,
        Phone.NUMBER},
    new int[]{
        android.R.id.text1,
        android.R.id.text2
    },
    0
);
```

Set a `FilterQueryProvider` so the adapter knows how to get content, given text to filter with. The query is run on a background thread, so we don't have to worry about blocking the main thread.

```Java
adapter.setFilterQueryProvider(new FilterQueryProvider() {
    @Override
    public Cursor runQuery(CharSequence constraint) {
        String constraintWithWildcard = constraint + "%";
        return getContentResolver().query(
            Data.CONTENT_URI,
            PROJECTION,
            SELECTION,
            new String[]{
                    constraintWithWildcard,
                    constraintWithWildcard
            },
            Contacts.DISPLAY_NAME_PRIMARY + " DESC"
        );
    }
});
```

We also need to provide a `CursorToStringConverter` so the adapter knows how to convert a contact row into a string the `AutoCompleteTextView` can use to complete a user's entry.

```Java
adapter.setCursorToStringConverter(new SimpleCursorAdapter.CursorToStringConverter() {
    @Override
    public CharSequence convertToString(Cursor cursor) {
        int phoneNumberIndex = cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER);
        return cursor.getString(phoneNumberIndex);
    }
});
```

All that's left is to set the adapter on the `AutoCompleteTextView` itself:

```Java
mRecipient = (AutoCompleteTextView) findViewById(R.id.recipient);
mRecipient.setAdapter(adapter);
```

Now that we can get contact info, let's see how we can open the Venmo app and pay a contact. Let's walk through a sample method to open Venmo and make a transaction.

```
private void doTransaction(String recipient, String amount, String note, String txn) {
    try {
        Intent venmoIntent = VenmoLibrary.openVenmoPayment(APP_ID, APP_NAME, recipient, amount, note, txn);
        startActivityForResult(venmoIntent, 1); //1 is the requestCode we are using for Venmo. Feel free to change this to another number.
    } catch (android.content.ActivityNotFoundException e) //Venmo native app not install on device, so let's instead open a mobile web version of Venmo in a WebView
    {
        Intent venmoIntent = new Intent(MyActivity.this, VenmoWebViewActivity.class);
        String venmo_uri = VenmoLibrary.openVenmoPaymentInWebView(APP_ID, APP_NAME, recipient, amount, note, txn);
        venmoIntent.putExtra("url", venmo_uri);
        startActivityForResult(venmoIntent, VENMO_REQUEST_CODE);
    }
}
```

We are using an `Intent`, a class Android provides to launch other components, to launch the Venmo app in order to complete a transaction. Note that if the app is not installed, a `WebView` will be launched that uses the Venmo website to complete the transaction. To get results from the App Switch SDK, we can define `onActivityResult` in our `Activity`, like so:

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data)
{
    switch(requestCode) {
        case 1: { //1 is the requestCode we picked for Venmo earlier when we called startActivityForResult
            if(resultCode == RESULT_OK) {
                String signedrequest = data.getStringExtra("signedrequest");
                if(signedrequest != null) {
                    VenmoResponse response = (new VenmoLibrary()).validateVenmoPaymentResponse(signedrequest, app_secret);
                    if(response.getSuccess().equals("1")) {
                        //Payment successful.  Use data from response object to display a success message
                        String note = response.getNote();
                        String amount = response.getAmount();
                    }
                }
                else {
                    String error_message = data.getStringExtra("error_message");
                    //An error occurred.  Display the error_message to the user.
                }                               
            }
            else if(resultCode == RESULT_CANCELED) {
                //The user cancelled the payment
            }
        break;
        }
    }
}
```

And that's it!
