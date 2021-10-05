# Contacts, Reborn!

Looking for the easiest way to get full-access to the breadth of Contacts in Android without having
to deal with the [Contacts Provider][contacts-provider] and cursors? Well, look no further =)

Whether you just need to get all or some Contacts for a small part of your app (written in Kotlin
or Java), or you are looking to create your own full-fledged Contacts app with the same capabilities
as the native Android Contacts app, this library has you covered!

For more context, read the [introductory, hype blog][medium-blog]!

The core library supports;
- All of the kinds of Data in the Contacts Provider; address, email, event, group membership, IM,
  name, nickname, note, organization, phone, photo, relation, SIP address, and website.
- Custom data.
- Broad and specific queries of Contacts and RawContacts from zero or more Accounts. Include only
  desired fields in the results (e.g. name and phone number) for optimization. Specify matching
  criteria in an SQL WHERE clause fashion using Kotlin infix functions. Order by contact table
  columns. Limit and offset functions.
- Insert one or more RawContacts with an associated Account, which leads to the insertion of a new
  Contact subject to automatic aggregation by the Contacts Provider.
- Update one or more Contacts, RawContacts, and Data.
- Delete one or more Contacts, RawContacts, and Data.
- Query, insert, update, and delete Profile (device owner) Contact, RawContact, and Data.
- Query, insert, and update Groups per Account.
- Query, update, and delete specific types of Data.
- Query, insert, update, and delete custom Data. 
- Query Accounts in the system or RawContacts table. 
- Associate local RawContacts (no Account) to an Account.
- Join/merge/link and separate/unmerge/unlink two or more Contacts.
- Get and set Contact and RawContact Options; starred (favorite), custom ringtone, send to voicemail.
- Get and set Contacts/RawContact photo and thumbnail.
- Get and set default (primary) Contact Data (e.g. default/primary phone number, email, etc).
- Miscellaneous convenience functions.

There are also extensions that add functionality to every core function;
- Asynchronous work using Kotlin Coroutines.
- Permissions request/handling using Kotlin Coroutines and Dexter.

Also included are some pre-baked goodies to be used as is or just for reference;
- Gender custom Data.
- Handle name custom Data.
- Rudimentary contacts-integrated UI components.

## Installation

First, include JitPack in the repositories list,

```
repositories {
    maven { url "https://jitpack.io" }
}
```

To import all modules,

```
dependencies {
     implementation 'com.github.vestrel00.contacts-android:0.1.0'
}
```

To import specific modules,

```
dependencies {
     implementation 'com.github.vestrel00.contacts-android:core:0.1.0'
}
```

SNAPSHOTs of branches are also available,

```
dependencies {
     implementation 'com.github.vestrel00.contacts-android:master-SNAPSHOT'
}
```

This library is a multi-module project published with JitPack. For all release artifacts published,
visit [![Release](https://jitpack.io/v/vestrel00/contacts-android.svg)](https://jitpack.io/#vestrel00/contacts-android).

For general JitPack project installations, visit https://jitpack.io/docs/BUILDING/#multi-module-projects

## Quick Start

To retrieve all contacts containing all available contact data,

```
Contacts(context).query().find()
```

To simply search for Contacts, yielding the exact same results as the native Contacts app,

```
Contacts(context)
    .broadQuery()
    .whereAnyContactDataPartiallyMatches(searchText)
    .find()
 ```

That's it! BUT, THAT IS BORING! Let's take a look at something more advanced…

To retrieve the first 5 contacts (including only the contact id, display name, and phone numbers in
the results) ordered by display name in descending order, matching ALL of these rules;
- a first name starting with "leo" 
- has emails from gmail or hotmail
- lives in the US
- has been born prior to making this query
- is favorited
- has a nickname of "DarEdEvil" (case sensitive)
- works for Facebook
- has a note
- belongs to the account of "jerry@gmail.com" or "jerry@myspace.com"

```
val contacts = Contacts(context)
    .query()
    .where(
        (Fields.Name.GivenName startsWith "leo") and
                ((Fields.Email.Address endsWith "gmail.com") or (Fields.Email.Address endsWith "hotmail.com")) and
                (Fields.Address.Country equalToIgnoreCase "us") and
                ((Fields.Event.Date lessThan Date()) and (Fields.Event.Type equalTo Event.Type.BIRTHDAY)) and
                (Fields.Contact.Options.Starred equalTo true) and
                (Fields.Nickname.Name equalTo "DarEdEvil") and
                (Fields.Organization.Company `in` listOf("facebook", "FB")) and
                (Fields.Note.Note.isNotNullOrEmpty())
    )
    .accounts(
        Account("jerry@gmail.com", "com.google"),
        Account("jerry@myspace.com", "com.myspace"),
    )
    .include(Fields.Contact.Id, Fields.Contact.DisplayNamePrimary, Fields.Phone.Number, Fields.Phone.NormalizedNumber)
    .orderBy(ContactsFields.DisplayNamePrimary.desc())
    .offset(0)
    .limit(5)
    .find()
```

Imagine what this would look like if you use ContactsContract directly. Now, you don't have to! The
above snippet is in Kotlin but, like I mentioned, all of the core APIs are usable in Java too
(though it won't look as pretty).

#### There's a lot more

This library is capable of doing more than just queries. Let's take a look at a few of them here.

To get the first 20 gmail emails ordered by email address in descending order,

```
val emails = Contacts(context).data()
    .query()
    .emails()
    .where(Fields.Email.Address endsWith  "gmail.com")
    .orderBy(Fields.Email.Address.desc(ignoreCase = true))
    .offset(0)
    .limit(20)
```

It's not just for emails. It's for all common data kinds (including custom data).

To create a contact with a name of "John Doe" who works at Amazon with a work email of
"john.doe@amazon.com" (in Kotlin),

```
val insertResult = Contacts(context)
    .insert()
    .rawContact {
        setName {
            givenName = "John"
            familyName = "Doe"
        }
        setOrganization {
            company = "Amazon"
            title = "Superstar"
        }
        addEmail {
            address = "john.doe@amazon.com"
            type = Email.Type.WORK
        }
    }
    .commit()
```

If John Doe switches jobs and heads over to Microsoft, we can update his data,

```
Contacts(context)
    .update()
    .contacts(johnDoe.toMutableContact().apply {
        setOrganization {
            company = "Microsoft"
            title = "Newb"
        }
        emails().first().apply {
            address = "john.doe@microsoft.com"
        }
    })
    .commit()
```

If we no longer like John Doe, we can remove  him from our life,

```
Contacts(context)
    .delete()
    .contacts(johnDoe)
    .commit()
```

#### There's even more…

This library provides Kotlin coroutine extensions for all API functions to handle permissions
(shoutouts to the Dexter team) and executing work in background threads.

```
launch {
    Contacts(context)
        .queryWithPermission()
        ...
        .findWithContext()
    Contacts(context)
        .insertWithPermission()
        ...
        .commitWithContext()
}
```

So, if we call the above function and we don't yet have permission. The user will be prompted to
give the appropriate permissions before the query proceeds. Then, the work is done in the coroutine
context of choice (default is Dispatchers.IO). If the user does not give permission, the query will
return no results.

Extensions for Kotlin Flow and RxJava is also in the v1 roadmap.

#### There's too much…

**The above examples barely scratches the surface of what this library provides.** For more in-depth
Howtos, visit the [howto directory](/howto/). For a sample app reference, take a look at and run the
`sample` module.

## Requirements

- Min SDK 19+
- Java 8+

## Proguard

If you use Proguard and the `async` and/or `permissions`, you may need to add rules for
[Coroutines][coroutines-proguard].

## License

    Copyright 2021 Contacts Contributors

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

## Support

This is a newly open sourced library with only one contributor so far (me). Don’t expect much
support in the beginning. I am only able to work on this (and respond to issues) outside of work
hours. This means late nights, weekends, and maybe holidays.

As time passes, hopefully this library gets more and more contributors. At some point, I hope to
gain contributors that have become experts on this library to help me support the community by
maintaining it and making admin-level decisions.

In any case, create issues for any bugs found and I'll get to it when I get the chance depending on
severity of the issue.

## v1.0.0 Release Roadmap

These are the remaining work that is required for this library to be considered worthy of being
v1.0.0 and safe for production.

To be clear, all core functions have been implemented and manually tested by me. As far as I'm
concerned, this library is production-ready as is, especially for daring/experimental consumers ;P
However, I cannot _professionally_ proclaim this to be production-ready without doing things that I
consider are essential for any API to do. So, here is my v1.0.0 release checklist!

- **(v0.1.0) Ready for public alpha development (current version)**
  - All v1 functions have been implemented and _manually tested_, with full documentation.
- **(v0.2.0) Sample app complete**
  - Purpose:
    - Prove that library produces same results as native Contacts app in actual usage
    - Prove that 0, 1, or more accounts are handled properly
    - Ensure all v1 functions are used in the sample app
    - All native Contacts app features are implemented in the sample app
      - Not including features that require post v1 functions
    - Find bugs in API
  - Use Pixel 3a API 30 native Contacts app as reference
  - Work required to be done for this version;
    - Create/edit/view contact screen
      - Select group memberships
      - Choose account to associate with the new (raw) contact (create-only)
      - Long press actions
        - Primary action (e.g. call phone number, send email, etc)
        - Copy to clipboard
        - Set default
    - Share contacts (export) (placeholder only - show toast saying it's coming post v1 release)
    - Link contacts
    - Create contact shortcut
    - Delete multiple contacts
    - Show all contacts (no filter)
    - Filter contacts by label (group memberships)
    - Create, update, delete non-system labels (group memberships)
    - Filter contacts by account
    - Settings
      - Profile create, read, update, delete
      - Show Accounts
      - Default account for new contacts
      - Contacts to display
      - Sort by first or last name
      - Name format
      - Phonetic name
      - Import (placeholder only - show toast saying it's coming post v1 release)
      - Export (placeholder only - show toast saying it's coming post v1 release)
      - Blocked numbers (placeholder only - show toast saying it's coming post v1 release)
      - About Contacts
- **(v0.3.0) Sample app enhanced**
  - Purpose:
    - Prove that additional (advanced) functions not in native Contacts app works
    - Find bugs in API
  - Work required to be done for this version;
    - Integrate Query (for more advance querying compared to BroadQuery).
    - Integrate DataQuery, DataUpdate, and DataDelete functions
    - Integrate handlename custom data
    - Integrate gender custom data
- **(v0.4.0) Sample app reuse**
  - Purpose:
    - Move/refactor as much code as possible from the sample app into reusable modules
    - Promote code reuse, including rudimentary code (that couldbe refined)
  - Work required to be done for this version;
    - Move view page to new module; ui-async
- **(0.5.0) Code quality and integrity**
  - Purpose:
    - Ensure that code follows language and community standards
    - Ensure that code is covered with automated tests
    - Prevent new code from breaking existing API
  - Work required to be done for this version;
    - androidTest/ for all core functions
    - Static analysis; lint / code quality checks (checkstyle, lint, etc)
    - Code coverage reporting tool
    - SonarQube?
- **(0.6.0) Kotlin Flow extensions**
  - Purpose:
    - Provide alternative asynchronous extensions for consumers prefer to use Flow over withContext
      and async await functions
    - Provide a way to proactively listen for changes in the Contacts database
  - Work required to be done for this version;
    - Add Flow function extensions in existing async module
    - Listen for changes in the Contacts database
- **(0.7.0) Reactive extensions**
  - Purpose:
    - Provide alternative asynchronous extensions for consumers that are not using Kotlin coroutines
    - Provide a way to proactively listen for changes in the Contacts database
  - Work required to be done for this version;
    - Create rx module
    - Implement rx-equivalent of async module functions
    - Listen for changes in the Contacts database
- **(0.8.0) Update dependencies and tools**
  - Purpose:
    - Ensure that the library and all of its dependencies and tools are up-to-date
  - Work required to be done for this version;
    - Update Kotlin stdlib and coroutines
      - Use sealed interface in Fields when upgrading to Kotlin 1.5+
    - Update Dexter
    - Update Gradle
    - Update Android Studio
- **(0.9.0) v1 release prep**
  - Purpose:
    - Ensure that the library is ready for v1.0.0 production release
  - Work required to be done for this version;
    - Setup continuous integration (CI/CD) pipeline (if not yet done)
    - Review remaining TODOs and FIXMEs, if any
    - Ensure all functions are covered with automated tests
    - Ensure all functions are documented with samples / howtos

## Post v1.0.0 Release Roadmap

These functions/features will not make the v1 release because I want to get the production-ready v1
version library out as soon as possible so that the community may benefit from it =)

Most, if not all, of these functions are the missing components required to rebuild the native
Android Contacts app from the ground up. In other words, each of these functions allow consumers to
implement a specific part of the native Android Contacts app.

1. Blocked phone numbers.
    - The Android 7.0 release introduces a BlockedNumberProvider content provider that stores a list
      of phone numbers the user has specified should not be able to contact them via telephony
      communications (calls, SMS, MMS).
    - See https://source.android.com/devices/tech/connect/block-numbers
2. SIM card query, insert(?), update(?), and delete(?).
    - Enables importing from and exporting(?) to SIM card.
    - Query will definitely be possible. I'm not sure if insert, update, and delete operations
      are possible. We will see.
3. Contacts read/write .VCF file.
    - Enables import from and export to .VCF file.
    - Enables sharing a contact.
    - Dev note: search ContactsContract for "vcard".
4. Social media custom data.
    - WhatsApp (vnd.com.whatsapp.profile)
    - Facebook Messenger
    - Twitter
    - Etc
    - I will need help from the community to implement these because I don't participate in
      social media. Use the `debug` module to look into the data table and look for social media
      mimetypes. Then, create a new custom data module using the existing `customdata-x` modules as
      reference. A separate module should be created for different social media.
      E.G. customdata-whatsapp, customdata-messenger, customdata-twitter
        - Read more in the DEV_NOTES "Custom Data / MimeTypes" section.

[medium-blog]: TODO
[contacts-provider]: https://developer.android.com/guide/topics/providers/contacts-provider
[coroutines-proguard]: https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-core/jvm/resources/META-INF/proguard/coroutines.pro
