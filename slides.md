# Improving Research through Advanced REDCap Interfaces

### Scott Burns

`scott.s.burns@vanderbilt.edu`

This talk can be found at:

- Source: `http://github.com/sburns/advanced-redcap-interfaces`)
- Current Version: `http://bit.ly/advanced-redcap`

## Presenter Notes

---

# REDCap is

<br />

## A data storage service

<br />

## Secure for the storage of PHI

<br />

## An online spreadsheet

## Presenter Notes

---

# REDCap is not

<br />

## ...a real database:

- Not relational, can only track one "entity" per project
- Each Project is only a single table

## ...a spreadsheet because

- Client (web-app) & Server (PHP/MySQL) architecture
- Only requires a browser
- Advanced user access controls
- Advanced web features (why we're here today)

## Presenter Notes

I actually believe the REDCap team made a great decision to not incorporate relational semantics to REDCap. They would have lost a lot of potential users.

IMO, REDCap should be judged not as a less capable database but as a super spreadsheet that insulates users from many of the pitfalls of file-based databases.

---

# Advanced REDCap Features

<br />

## Application Programming Interface

External software can push and pull data over the internet

## Data Entry Triggers

Saving any data causes HTTP POST to URL (of your choosing)

**These features are the building blocks of extremely advanced data managment techniques and they're ready to be used right now.**

## Presenter Notes

**These two features make REDCap the foundation for advanced data management workflows**

Investing in these workflows will improve your work. The techincal details of how these two features work are less important than the workflows they enable, which I'll talk about towards the end.

---

# Advanded data manag...what?

<br />

**Ideally, machines peform all trivial data analysis because they:**

*   Perform reproducible work.
*   Are extremely cheap.
*   Operate deterministically.

## Presenter Notes

As opposed to humans who:

*   Cannot be relied upon to reproduce their own work.
*   Are extremely expensive.
*   Some RAs might even be considered pseudorandom processes.

---

# Vocabulary

<br />

<table>
    <thead>
        <tr>
            <th class="left"><strong>REDCap</strong></th>
            <th><strong>SQL/Excel/etc</strong></th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td class="left">Project</td>
            <td>Table</td>
        </tr>
        <tr>
            <td class="left">Data Dictionary</td>
            <td>Schema</td>
        </tr>
        <tr>
            <td class="left">Record</td>
            <td>Row</td>
        </tr>
        <tr>
            <td class="left">Field</td>
            <td>Column</td>
        </tr>
        <tr>
            <td class="left">Form or Instrument</td>
            <td>Set of columns</td>
        </tr>
        <tr>
            <td class="left">Unique Identifier</td>
            <td>Primary Key, Index, etc</td>
        </tr>

<!--         <tr>
            <td class="left"></td>
            <td></td>
        </tr>
 -->    </tbody>
</table>

---

# API


HTTP `POST` -> `https://redcap.vanderbilt.edu/api/`

`https://redcap.vanderbilt.edu/api/help/` describes payload parameters

## Note:

-   You need API access to the Project and a generated token
-   In-/output formats: `csv`, `xml`, `json`

## Presenter Notes

HTTP-based API for communicating with Projects

All API calls against REDCap are basically the same thing.

The API token binds your user rights to a specific project. DO NOT SHARE THIS TOKEN.

JSON (Javascript Object Notation) is the most interesting format from my perspective because it can be decoded to real objects.

---

# API

## Any language that has an HTTP library can communicate with REDCap.

**PyCap** is a python package I wrote and maintain to facilitate REDCap API usage within python applications/scripts.

`$ pip install PyCap`

`https://sburns.github.com/PyCap` for documentation.


## Presenter Notes

All languages worth using have an http library.

I'm going to talk about things implemented in python, but it's important to know that nothing about the REDCap API requires you to use python.

While I'm talking about the API, I'm going to show curl commands towards the top and the corresponding python calls below.

---

# API

## Most Important Methods

*   Export Data-Dictionary
*   Export & Import Records
*   Export/Import/Delete Files

---

# API: Exporting Data-Dictionary


-   Name
-   Label (Human Readable)
-   Type (Dropbown, Text, Textbox, etc)
-   Choices
-   Branching Logic (If any)
-   And others

Response is equivalent to downloading the data dictionary through the web application.

## Presenter Notes

A table about your table. Useful to determine whether fields exist before imports/etc.


---

# API: Exporting the Data-Dictionary


    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=metadata \
        -d format=json

<br />

    !python
    from redcap import Project
    project = Project('https://redcap.vanderbilt.edu/api/', TOKEN)
    md = project.export_metadata()


## Presenter Notes

---

# API: Exporting Records

**Download the entire database**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv

<br />

    !python
    data = project.export_records()
    # data is a list of dictionary objects with one dictionary per record
    csv = project.export_records(format='csv') # or 'xml'
    df = project.export_records(format='df')  # Pandas DataFrame

## Presenter Notes

Getting a DataFrame is really helpful because pandas automatically provides type coercion in the DF construction (a text field column that stores numbers -> floats)

---

# API: Exporting Data

**Or request slices**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv \
        -d records=1,2,3 \
        -d fields=age,sex,gender

<br />

    !python
    sliced = project.export_records(records=['1', '2', '3'],
                                    fields=['age', 'sex', 'gender'])


## Presenter Notes

If you're working with a big database, you can speed up the API call by requesting only rows and columns that you'll need.

---

# API: Importing Data

**Update existing records or add new ones on the fly**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv \
        -d data="participant_id,new_data\n1,hooray"

<br />

    !python
    project.import_records([{'participant_id': '1', 'new_data': 'hooray'}])
    # Or upload many records at once
    from my_module import modify_records
    data = project.export_records()
    modified = modify_records(data)
    response = project.import_records(modified)
    assert response['count'] == len(modified)  # Just to make sure

## Presenter Notes

It's most likely best to download data, tweak it and import.

By default does not overwrite non-imported fields with empty data.

---

# API: File Exporting

**Works on a per-record basis**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/
        -d token=XXX \
        -d returnFormat=json \
        -d content=file \
        -d action=export \
        -d record=1 \
        -d field=file > exported_file.txt

<br />

    !python
    content, headers = project.export_file(record='1', field='file')
    # write out a new file using the filename as stored in REDCap
    with open(headers['name'], 'w') as f:
        f.write(content)


---

# API: File Importing

**Works on a per-record basis**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/
        -d token=XXX \
        -d returnFormat=json \
        -d content=file \
        -d action=import \
        -d record=1 \
        -d field=file \
        -d file=@localfile.txt

<br />

    !python
    local_fname = 'to_upload.pdf'
    with open(local_fname, 'rb') as fobj:
        response = project.import_file(record='1', field='file',
                                       fname=local_fname, fobj=fobj)
    # Whatever is passed to the fname argument will appear in the REDCap UI

---

# API: File Deletion

**Works on a per-record basis**

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/
        -d token=XXX \
        -d returnFormat=json \
        -d content=file \
        -d action=delete \
        -d record=1 \
        -d field=file \

<br />

    !python
    project.delete_file('1', 'file')

---

# API: Example Usage

<br />
## Advanced & Automated Field Calculations

<br />

## REDCap as an interface for other systems/processes of research

<br />

## Shared filesystem

<br />

---

# API: Advanced Field Calculations

## Worst Implementation

*   Download the data as spreadsheet/SPSS/etc.
*   Implement the field calculation and execute.
*   Re-upload data.

**Why this is bad**

*   Must re-do the process for each new record.
*   Can you trust who ever does this to always do it perfectly?
*   Delete the spreadsheet and a piece of the methods section potentially goes to the trash.
*   Always the same calculation or are we introducing bias?

---

# API: Advanced Field Calculations

## Better Implementation

**While REDCap does provide a calculated field feature, their implementation has issues**

*   Implemented as Javascript, requires viewing the field in the web app for the calculation to execute.
*   No access to third-party code (statistics/conversion tables/etc)
*   Must re-implement across Projects.

## Presenter Notes

What if you have hundreds of records? Some poor soul has to click a button that many times.


# API: Advanced Field Calculations


**If we write our field calculations against the API**

*   Write the (testable!) implementation once
*   Easily use against many projects
*   Fast, Error-Free IO
*   Upfront cost amortized across all automated calculations

## Presenter Notes

Software testing in academia is for another talk, but do you really want to publish using untested methodologies?

Using the API, we can write and test advanced field calculations once and apply them across projects easily.

While the calculation itself is probably no faster than when a human does it, we save time, energy and reduce mistakes in the download/upload process.

---

# API: As an Interface to Other Systems

A major thrust of our work is neuroimaging in children.

We use REDCap as an interface for our image processing backend that runs at ACCRE.

Imaging Data is record-aligned with behavioral data we store in REDCap.

## Presenter Notes

This is difficult to explain/talk about generally, but REDCap did a lot of hard work in making a nice UI for humans to input data.

The API presents a stationary target that other systems can be written against to grab information and use it to perform "business logic".

---

# API: Other processes of research

**Come analysis time, the API can help in these ways and more:**

## Reproducible group/cohort determination

## Automated database cleanup

## TODO

## Presenter Notes

Reduces Friction between researchers and their data.

Use the same analyses for pilot and production data.

Theoretically, your research software that uses the REDCap API is documented, tested and should help write the Methods sections.

---

# API: A Shared Filesystem

<br />

Research often produces intermediary data that needs to be processed, the results of which should be stored in REDCap.

How do we connect intermediate files to our analysis infrastructure?

REDCap `file` fields!

---

# API: A Shared Filesystem

**A general approach**

*   Lab member runs test, uploads intermediate file to specific field
*   Automated program exports file to local filesystem
*   Automated program analyses and uploads results to REDCap

**No need to share a filesystem**

*   Disconnect an unsecure environment (the lab member) from an environment with potential PHI (where the analysis runs).
*   The automated program can also organize files better/faster/cheaper than any human.

---

# API: A Shared Filesystem

**What about the other direction?**

Given a record in the database:

*   An automated program can produce a novel file based on the record
*   Upload it to REDCap
*   Alert lab members of the file creation.

## Presenter Notes

In our lab and many labs that test children, the "carrot stick" for convincing parents to cooperate is to send out reports about their child's scores on behavioral tests.

Using the data export and file upload methods, we can create these reports automatically and store the resulting file in REDCap so other lab members have access to them.

---

# API: A final thought

<br />
<br />

**What the API can do for your research is up to you**

## Presenter Notes

I've only scratched the surface here. The possibilities of workflows enabled by the API are essentially limitless.

---

# API: Pitfalls

**Accessing the API is solely driven by external requests**

**You could poll the Project for new data, but that's inefficient and computationally wasteful**

**Can we have a better idea about when to run analyses?**

---

# Data Entry Triggers

<br />

## Independent but Complimentary Feature to the API

*   Register a URL to your Project
*   ANY Form save --> HTTP POST request to URL.

---

# Data Entry Triggers

<br />

<img src="img/animated-fireworks-wallpaper.jpeg" width="640" height="400">
