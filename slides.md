# Improving Research through Advanced REDCap Interfaces

### Scott Burns

`scott.s.burns@vanderbilt.edu`

This talk can be found at:

`http://github.com/sburns/advanced-redcap-interfaces`
`http://bit.ly/advanced-redcap`

Notes & Code Samples in the handouts


## Presenter Notes

I'm an open book, this talk can be found on my github.

A rendered version can be found at the bit.ly URL.

Ratio of coders to non-coders?

The handouts have my talking points. Where applicable, they also contain code samples. I hope you find this useful.

---

# Goals

<br />

## REDCap provides advanced features but doesn't talk about them.

<br />

## These features make possible advanced data management workflows.

<br />

## I will show you the way.

## Presenter Notes

I have no affiliation with REDCap other than I submit a lot of bug reports, though less recently which is a good thing.

If your head isn't swimming with ideas about how to improve your research, I haven't done my job.

---

# Advanded data manag...what?

<br />

## In an ideal world

Machines peform all **definable** data analyses because they:

*   Perform reproducible work.
*   Are extremely cheap.
*   Operate deterministically.

## Presenter Notes

As opposed to humans who:

*   Cannot be relied upon to reproduce their own work.
*   Are extremely expensive.
*   Some RAs might even be considered pseudorandom processes.

---

# REDCap is...

<br />

## ...a data storage service

<br />

## ...secure for the storage of PHI

<br />

## ...an online spreadsheet

## Presenter Notes

Just to get everyone on the same page

---

# REDCap is less than...

<br />

## ...a "real" database:

- Not relational, can only track one "entity" per project
- Each Project is only a single table

<img src="img/db-comp.png" width="480">

## Presenter Notes

REDCap does not support traditional database features as found RDBMS like Oracle, MySQL, etc.

Namely, it is not relational. A REDCap project can only track single "entities".

Other databases let you add fancy relationships (one-to-one, one-to-many) to your models.

This decision reduces a lot more complexity and I applaud them for it.

---

# REDCap is more than...

<br />

## ...a spreadsheet because

- Client & Server architecture
- Browser-based.
- User access controls
- Advanced web features (why we're here today)

## Presenter Notes

Web application is a front-end to a large server backend.

No installation required on users end.

IMO, REDCap should be judged not as a less capable database but as a super spreadsheet that insulates users from many of the pitfalls of file-based tabular databases.

We're here today to talk about these advanced web features

---

# Advanced REDCap Features

<br />

## Application Programming Interface

External software can push and pull data.

## Data Entry Triggers

Internet notifications for anytime data is saved.

**These features are the building blocks of extremely advanced data managment techniques and they're ready to use right now.**

## Presenter Notes

**These two features make REDCap the foundation for advanced data management workflows**

Investing in these workflows will improve your work. The techincal details of how these two features work are less important than the workflows they enable, which I'll talk about towards the end.

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

# General Architecture

<br />
<img src="img/arch-total.png" width="720">

---

# General Architecture (REDCap)

<br />
<img src="img/arch-redcap.png" width="720">

---

# General Architecture (You)

<br />
<img src="img/arch-lab.png" width="720">

---

# API

<br />
<img src="img/api.png" width="720">

---

# API

## HTTP POST to API URL

`https://redcap.vanderbilt.edu/api/`

## Any language that has an HTTP library can communicate with REDCap.

**PyCap** facilitates using the REDCap API within python applications.

(`https://sburns.github.com/PyCap`)


## Presenter Notes


HTTP-based API for communicating with Projects

All languages worth using have an http library.

In-/output formats: `csv`, `xml`, `json`

You'll find code snippets on the handouts

*   `$ pip install PyCap`
*   `https://redcap.vanderbilt.edu/api/help/`

---

# API

<br />

## Most Important Methods

*   Export Data-Dictionary (rules for your database/schema/etc)
*   Export & Import Records (Download/importing spreadsheets)
*   Export/Import/Delete Files

---

# API: Exporting Data-Dictionary

Completely describes your database design

## For each column:

-   Field Name
-   Field Label (Human Readable)
-   Type (Dropbown, Text, Textbox, etc)
-   Choices
-   Branching Logic (if any)
-   And others

Equivalent to downloading the data dictionary through the web application.

## Presenter Notes

A table about your table. Useful to determine whether fields exist before imports/etc.

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

---

# API: Exporting Records

<br />

## Download the entire database

<br />

## Can also request certain columms or records

<br />

## Honors access rules

## Presenter Notes

Grab all of the data or just some

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv
    $ curl -X POST https://redcap.vanderbilt.edu/api/ \
        -d token=XXX \
        -d content=record \
        -d format=csv \
        -d records=1,2,3 \
        -d fields=age,sex,gender

<br />

    !python
    data = project.export_records()
    # data is a list of dictionary objects with one dictionary per record
    csv = project.export_records(format='csv') # or 'xml'
    df = project.export_records(format='df')  # Pandas DataFrame
    sliced = project.export_records(records=['1', '2', '3'],
                                    fields=['age', 'sex', 'gender'])

Getting a DataFrame is really helpful because pandas automatically provides type coercion in the DF construction (a text field column that stores numbers -> floats)

---

# API: Importing Records

<br />
## Update existing records

<br />

## Add new records on the fly

<br />

## Hard to overwrite data

## Presenter Notes

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

---

# API: File Actions

<br />

## Export (Download)

<br />
## Import (Upload)

<br />
## Delete

## Presenter Notes

Download

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


Importing

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/
        -d token=XXX \
        -d returnFormat=json \
        -d content=file \
        -d action=import \
        -d record=1 \
        -d field=file \
        -d file=@localfile.txt


    !python
    local_fname = 'to_upload.pdf'
    with open(local_fname, 'rb') as fobj:
        response = project.import_file(record='1', field='file',
                                       fname=local_fname, fobj=fobj)
    # Whatever is passed to the fname argument will appear in the REDCap UI

Deleting

    !bash
    $ curl -X POST https://redcap.vanderbilt.edu/api/
        -d token=XXX \
        -d returnFormat=json \
        -d content=file \
        -d action=delete \
        -d record=1 \
        -d field=file \

    !python
    project.delete_file('1', 'file')

---

# API: Possible Uses

<br />
## Advanced & Automated Field Calculations

<br />

## REDCap as an interface for other systems/processes of research

<br />

## Shared filesystem

<br />

---

# API: Advanced Field Calculations


## Problem: Update Many Records By Applying Formula

<img src="img/bad.png" width="720">

## Presenter Notes

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


## Problem: Update Many Records By Applying Formula


<img src="img/better.png" width="720">


## Presenter Notes

**While REDCap does provide a calculated field feature, their implementation has issues**

*   Implemented as Javascript, requires viewing the field in the web app for the calculation to execute.
*   No access to third-party code (statistics/conversion tables/etc)
*   Must re-implement across Projects.


What if you have hundreds of records? Some poor soul has to click a button that many times.

---

# API: Advanced Field Calculations


## Problem: Update Many Records By Applying Formula

<img src="img/best.png" width="720">

## Presenter Notes


*   Write the (testable!) implementation once
*   Easily use against many projects
*   Fast, Error-Free IO
*   Upfront cost amortized across all automated calculations

Software testing in academia is for another talk, but do you really want to publish using untested methodologies?

Using the API, we can write and test advanced field calculations once and apply them across projects easily.

While the calculation itself is probably no faster than when a human does it, we save time, energy and reduce mistakes in the download/upload process.

---

# API: External Research Systems

<br />
## External Processing/Databases

<br />
## Reproducible group/cohort determination

<br />
## Automated database cleanup


## Presenter Notes

*   A major thrust of our work is neuroimaging in children.
*   We use REDCap as an interface for our image processing backend that runs at ACCRE.
*   Imaging Data is record-aligned with behavioral data we store in REDCap.


This is difficult to explain/talk about generally, but REDCap did a lot of hard work in making a nice UI for humans to input data.

The API presents a stationary target that other systems can be written against to grab information and use it to perform "business logic".

Reduces Friction between researchers and their data.

Use the same analyses for pilot and production data.

Theoretically, your research software that uses the REDCap API is documented, tested and should help write the Methods sections.

---

# API: A Shared Filesystem

<br />

## How do we connect *intermediate* data to our analysis infrastructure?

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

# API

<br />

<img src="img/clouds.jpg" width="720">

## Presenter Notes

I've only scratched the surface here. The possibilities of workflows enabled by the API are essentially limitless.

---

# API: Downfall

<br />

## API usage is driven by external requests

Poll? (yuck)

**Can we get a better idea about when to run analyses?**

---

# Data Entry Triggers

<br />
<img src="img/det.png" width="720">
---

# Data Entry Triggers

<br />

## Independent but Complimentary Feature to the API

*   Register a URL to your Project
*   ANY Form save --> HTTP POST request to URL.

Picture

---

# Data Entry Triggers

<br />

<img src="img/animated-fireworks-wallpaper.jpeg" width="640" height="400">

---

# Data Entry Triggers

<img src="img/det-fireworks.png" width="720">

## Presenter Notes

*   Deploy arbitrary data workflows on a webserver
*   Workflows execute in real time as data is saved.
*   Otherwise normal users can execute very advanced processing.
*   "It just happens"

---

# Data Entry Triggers: Incoming Payload

<img src="img/det-total.png" width="720">


## Presenter Notes

**Fields in the incoming payload:**

*   `project_id`: Project Identifier
*   `instrument`: What form was saved?
*   `record`: Which record?
*   `redcap_event_name`: For which event (longitudinal projects only)
*   `redcap_data_access_group`: What "kind" of user saved it?
*   `[instrument]_complete`: What is the status of the form?

Fake the incoming payload, blast off **many** analyses.

---

# Data Entry Triggers: Pitfalls

<br />

## Not every research group...

*   ...can setup/maintain/secure a webserver.
*   ...has the resources to write the web application.

## But every lab should have access to this functionality

## Presenter Notes

(Middleware is required to route incoming requests to the correct workflow)


---

# Data Entry Triggers: Switchboard

`switchboard`:

*   Parses incoming `POST` requests from REDCap (or whomever).
*   Executes only the functions whose workflows match the request.

**In Production** for our lab but **rough around the edges**


<br />
<br />

(`github.com/sburns/switchboard` want to help?)

---

# Data Entry Triggers

<br />

## In a perfect world...

A shared "switchboard" webserver:

*   Just one webserver to maintain & protect.
*   Shared infrastructure is good.
*   Decrease "activation energy" for groups to use these features.
*   Optimize pieces and all groups benefit.

---

# Automating machines easier than humans

<img src="img/api.png" width="720">

---

# Automate the automation!

<img src="img/det-total.png" width="720">

---

# Thank You

<br />

<img src="img/cutting.png" width="200">
<img src="img/davis.png" width="200">
<img src="img/rimrodt.png" width="200">

---

# Questions?