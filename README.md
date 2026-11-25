# DEVELOPING-AN-ADDRESS-BOOK-ANDROID-APPLICATION
To design and develop an Address Book Android application using Android Studio

# AIM
To design and develop an Address Book Android application using Android Studio that allows users to add, store, view and manage contact details such as name, phone number and email address through a user-friendly graphical interface.

# SOFTWARE REQUIREMENTS
•	Android Studio
•	Java Programming Language
•	XML Layout Editor
•	Android Emulator or Android Mobile
•	Android SDK

# THEORY
Android Studio is an Integrated Development Environment (IDE) used for developing Android applications using Java and XML.
An Address Book application is used to maintain and manage personal contact information. It provides a graphical user interface for entering and displaying contact details.
The application uses the following Android UI components:
•	EditText: To enter the contact name, phone number and email address.
•	Button: To add and save contact details.
•	TextView: To display contact information.
•	LinearLayout: To arrange the user interface components.
•	ListView: To display the list of saved contacts.
Java event-handling methods are used to process user inputs and perform contact management operations.

# PROCEDURE
1.	Open Android Studio.
2.	Select New Project and choose Empty Views Activity.
3.	Enter the project name as AddressBook.
4.	Select Java as the programming language.
5.	Design the user interface using the XML Layout Editor.
6.	Add EditText fields to enter the contact name, phone number and email address.
7.	Add a Button to save the contact details.
8.	Add a ListView to display the saved contacts.
9.	Open MainActivity.java and write the Java program to process contact information.
10.	Use setOnClickListener() to handle the Save Contact button event.
11.	Store the entered contact details in an appropriate data structure.
12.	Display the saved contacts in the ListView.
13.	Execute the application using an Android Emulator or physical Android device.
14.	Enter contact details and click the Save Contact button.
15.	Verify that the contact information is displayed successfully.

# ALGORITHM
1.	Start the application.
2.	Initialize the Android user interface components.
3.	Accept the contact name from the user.
4.	Accept the phone number and email address.
5.	Validate the entered contact details.
6.	Click the Save Contact button.
7.	Store the contact information.
8.	Display the saved contact in the contact list.
9.	Repeat the process to add additional contacts.
10.	Stop
# PROGRAM
XML
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/nameInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Name"
        android:inputType="textPersonName" />

    <EditText
        android:id="@+id/phoneInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Phone"
        android:inputType="phone" />

    <EditText
        android:id="@+id/emailInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Email"
        android:inputType="textEmailAddress" />

    <EditText
        android:id="@+id/addressInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Address"
        android:inputType="textPostalAddress" />

    <Button
        android:id="@+id/saveButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="Save Contact" />

    <ListView
        android:id="@+id/contactList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="16dp" />

</LinearLayout>

KT
package com.example.contact

import android.database.Cursor
import android.os.Bundle
import android.view.ViewGroup
import android.widget.ArrayAdapter
import android.widget.Button
import android.widget.EditText
import android.widget.LinearLayout
import android.widget.ListView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    private lateinit var nameInput: EditText
    private lateinit var phoneInput: EditText
    private lateinit var emailInput: EditText
    private lateinit var addressInput: EditText
    private lateinit var saveButton: Button
    private lateinit var contactList: ListView

    private lateinit var db: DatabaseHelper
    private val contactIds = ArrayList<Int>()
    private val contacts = ArrayList<String>()
    private lateinit var adapter: ArrayAdapter<String>

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        nameInput = findViewById(R.id.nameInput)
        phoneInput = findViewById(R.id.phoneInput)
        emailInput = findViewById(R.id.emailInput)
        addressInput = findViewById(R.id.addressInput)
        saveButton = findViewById(R.id.saveButton)
        contactList = findViewById(R.id.contactList)

        db = DatabaseHelper(this)

        adapter = ArrayAdapter(
            this,
            android.R.layout.simple_list_item_1,
            contacts
        )
        contactList.adapter = adapter

        saveButton.setOnClickListener { saveContact() }

        contactList.setOnItemClickListener { _, _, position, _ ->
            editContact(position)
        }

        contactList.setOnItemLongClickListener { _, _, position, _ ->
            confirmDelete(position)
            true
        }

        loadContacts()
    }

    private fun saveContact() {
        val name = nameInput.text.toString().trim()
        val phone = phoneInput.text.toString().trim()
        val email = emailInput.text.toString().trim()
        val address = addressInput.text.toString().trim()

        if (name.isEmpty() || phone.isEmpty()) {
            Toast.makeText(this, "Enter name and phone number", Toast.LENGTH_SHORT).show()
            return
        }

        val result = db.addContact(name, phone, email, address)

        if (result) {
            Toast.makeText(this, "Contact saved", Toast.LENGTH_SHORT).show()
            clearFields()
            loadContacts()
        } else {
            Toast.makeText(this, "Unable to save contact", Toast.LENGTH_SHORT).show()
        }
    }

    private fun loadContacts() {
        contacts.clear()
        contactIds.clear()

        val cursor: Cursor = db.getContacts()
        try {
            while (cursor.moveToNext()) {
                val id = cursor.getInt(cursor.getColumnIndexOrThrow("id"))
                val name = cursor.getString(cursor.getColumnIndexOrThrow("name"))
                val phone = cursor.getString(cursor.getColumnIndexOrThrow("phone"))
                val email = cursor.getString(cursor.getColumnIndexOrThrow("email"))
                val address = cursor.getString(cursor.getColumnIndexOrThrow("address"))

                contactIds.add(id)
                contacts.add("$name\nPhone: $phone\nEmail: $email\nAddress: $address")
            }
        } finally {
            cursor.close()
        }

        adapter.notifyDataSetChanged()
    }

    private fun clearFields() {
        nameInput.setText("")
        phoneInput.setText("")
        emailInput.setText("")
        addressInput.setText("")
        nameInput.requestFocus()
    }

    private fun editContact(position: Int) {
        val id = contactIds[position]
        val cursor: Cursor = db.getContacts()
        try {
            while (cursor.moveToNext()) {
                if (cursor.getInt(cursor.getColumnIndexOrThrow("id")) == id) {
                    val name = cursor.getString(cursor.getColumnIndexOrThrow("name"))
                    val phone = cursor.getString(cursor.getColumnIndexOrThrow("phone"))
                    val email = cursor.getString(cursor.getColumnIndexOrThrow("email"))
                    val address = cursor.getString(cursor.getColumnIndexOrThrow("address"))

                    showEditDialog(id, name, phone, email, address)
                    return
                }
            }
        } finally {
            cursor.close()
        }
    }

    private fun showEditDialog(id: Int, name: String, phone: String, email: String, address: String) {
        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setPadding(32, 8, 32, 0)
        }

        val editName = makeField("Name", name)
        val editPhone = makeField("Phone", phone)
        val editEmail = makeField("Email", email)
        val editAddress = makeField("Address", address)

        layout.addView(editName)
        layout.addView(editPhone)
        layout.addView(editEmail)
        layout.addView(editAddress)

        val dialog = AlertDialog.Builder(this)
            .setTitle("Edit Contact")
            .setView(layout)
            .setPositiveButton("Update", null)
            .setNegativeButton("Cancel", null)
            .create()

        dialog.setOnShowListener {
            dialog.getButton(AlertDialog.BUTTON_POSITIVE).setOnClickListener {
                val newName = editName.text.toString().trim()
                val newPhone = editPhone.text.toString().trim()

                if (newName.isEmpty() || newPhone.isEmpty()) {
                    Toast.makeText(this, "Name and phone are required", Toast.LENGTH_SHORT).show()
                    return@setOnClickListener
                }

                val updated = db.updateContact(
                    id,
                    newName,
                    newPhone,
                    editEmail.text.toString().trim(),
                    editAddress.text.toString().trim()
                )

                if (updated) {
                    loadContacts()
                    Toast.makeText(this, "Contact updated", Toast.LENGTH_SHORT).show()
                    dialog.dismiss()
                } else {
                    Toast.makeText(this, "Update failed", Toast.LENGTH_SHORT).show()
                }
            }
        }

        dialog.show()
    }

    private fun makeField(hint: String, value: String): EditText {
        return EditText(this).apply {
            this.hint = hint
            setText(value)
            layoutParams = LinearLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT
            )
        }
    }

    private fun confirmDelete(position: Int) {
        val id = contactIds[position]

        AlertDialog.Builder(this)
            .setTitle("Delete Contact")
            .setMessage("Do you want to delete this contact?")
            .setPositiveButton("Delete") { _, _ ->
                if (db.deleteContact(id)) {
                    loadContacts()
                    Toast.makeText(this, "Contact deleted", Toast.LENGTH_SHORT).show()
                } else {
                    Toast.makeText(this, "Delete failed", Toast.LENGTH_SHORT).show()
                }
            }
            .setNegativeButton("Cancel", null)
            .show()
    }

    override fun onDestroy() {
        db.close()
        super.onDestroy()
    }
}


# EXPECTED OUTPUT
ADDRESS BOOK APPLICATION
<img width="1872" height="1016" alt="Screenshot 2026-11-25 092338" src="https://github.com/user-attachments/assets/478cfb37-e8e5-49d8-8b85-800f779c5e03" />


# APPLICATION FEATURES
•	User-friendly graphical interface.
•	Add new contact details.
•	Store name, phone number and email address.
•	Display saved contacts in a list.
•	Validate user-entered information.
•	Manage multiple contact entries.

# RESULT
Thus, the Address Book Android application was successfully designed and developed using Android Studio with Java and XML. The application allows users to enter, save and display contact details successfully.

# CONCLUSION
The experiment provides practical knowledge of Android application development, XML-based user interface design, Java programming, event handling and contact management using Android Studio.

