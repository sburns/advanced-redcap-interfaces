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

- Not relational, can only track one "entity"
- Each Project is only a single table

## ...a spreadsheet because

- Web-, not file-based
- Only requires a browser
- Advanced user access controls
- TODO

---

# Advanced REDCap Features

<br />

## Application Programming Interface

External software can push and pull data over the internet

## Data Entry Triggers

Saving any data --> HTTP POST to URL (of your choosing)


## Presenter Notes

**These two features make REDCap the foundation for advanced data management workflows**
Investing in these workflows will improve your work. Will provide examples at the end.

---

# So what?

<br />

**Ideally, machines peform all trivial data analysis because:**

*   Create reproducible work.
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

HTTP-based API for communicating with Projects

`https://redcap.vanderbilt.edu/api/`

Any language that implements an HTTP library can communicate with REDCap.

`https://redcap.vanderbilt.edu/api/help`


## Presenter Notes

All languages worth using have an http library.

I'm going to talk about things implemented in python, but it's important to know that nothing about the REDCap API requires you to use python.

---

# API

## Most Important Methods

*   Export Data-Dictionary
*   Export Records
*   Import Records
*   Export/Import/Delete Files

---

# API

## Usage

Make an HTTP `POST` request with a correctly-formatted payload.

Example:

`curl -X POST https://redcap.vanderbilt.edu/api/ -d token=XXX -d content=metadata -d format=json`

