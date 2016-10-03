# GedcomParse
Parse JSON data out of a .gedcom file using gedcompy & python2.7

>TODO:
  >>get family relations (currently only gets person data)
  
## Installation:
### Python
<a href="http://www.pyladies.com/blog/Get-Your-Mac-Ready-for-Python-Programming/"> Mac Installation </a> <br>
<a href="http://www.howtogeek.com/197947/how-to-install-python-on-windows/"> Windows Installation </a> <br>
<a href="http://docs.python-guide.org/en/latest/starting/install/linux/"> Linux Installation </a>
### GedcomPy
DO NOT clone from the gedcompy git, I have made changes for this use of the library

###### Installation:

navigate into the gedcompy folder and install:

  `cd gedcompy` <br>
  `python setup.py build` <br>
  `python setup.py install` - may have to use sudo

### GedcomParse
##### For Local use:
Clone from this repository & move any .ged files into the GedComParse folder to be parsed into json.
##### Requirements
* `python2.7` : see above
* <a href='https://pypi.python.org/pypi/pip'>`pip`</a> (package installer python)
* <a href='https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior'>`datetime`</a> 
* `gedcompy` : see below

##### Node Modules
* `child-process` - running python from node
* <a href='https://github.com/expressjs/multer'>`multer`</a> - for uploading files
* `body-parser`
* `express`
* `mongoose`


  
## Usage (from terminal):
* Move the gedcom file into the GedComParse folder for use.

Usage: `python gedcomparse.py [-h] INPUT_FILE.ged OUTPUT_FILE.json`

> Parse a .gedcom file into .json for use in FamilyGenie

## Usage (from Node server):
```javascript
// multer options
var upload = multer({ dest : 'uploads/' });
// executable child process:
var exec = require('child_process').exec;
//run the process on post
app.post('/uploads', upload, 
    function(req,res,next) {
        exec('python //run python
        path/to/gedcomparse.py //on this program
        path/to/gedcom-file.ged //using this file
        path/to/new-json-file.json', //to this file
        function(err) {
            if(err) { 
                console.log(err); 
            }
            // then to put the files into mongo ...
            exec('mongoimport 
                --db database 
                --collection collection 
                --type json 
                --file path/to/json-file.json 
                --jsonArray', 
                function(err) {
                    if(err) {
                        console.log(err);
                    }
            });
        });
    res.redirect('/');
});

```

## gedcompy Usage:
gedcompy parses out the records of each person and stores them as nested objects, accesible through dot notation.

Example Usage
-------------
```python
    >>> import gedcom
    >>> gedcomfile = gedcom.parse("myfamilytree.ged")
    >>> for person in gedcomfile.individuals:
    ...    firstname, lastname = person.name
    ...    print "{0} {1} is in the file".format(firstname, lastname)
```

The file as a whole is a generator object.

```python
>>> import gedcom
>>> gedfile = gedcom.parse("myfamilytree.ged")
>>> print gedfile
GedcomFile(
Element(0, 'HEAD', [Element(1, 'CHAR', 'UTF-8'), Element(1, 'SOUR', 'Ancestry.com Family Trees', [Element(2, 'VERS', '(2010.3)'), Element(2, 'NAME', 'Ancestry.com Family Trees'), Element(2, 'CORP', 'Ancestry.com')]), Element(1, 'GEDC', [Element(2, 'VERS', '5.5'), Element(2, 'FORM', 'LINEAGE-LINKED')])]),
Individual(0, 'INDI', '@P1@', [Birth(1, 'BIRT', [Element(2, 'DATE', '03 dec 1970')]), Element(1, 'SEX', 'M'), Element(1, 'NAME', 'John /Smith/'), Element(1, 'FAMC', '@F1@')])
Individual(0, 'INDI', '@P2@', [Element(1, 'NAME', 'Jane /Doe/'), Element(1, 'SEX', 'M'), Birth(1, 'BIRT', [Element(2, 'DATE', '06 nov 1946'), Element(2, 'PLAC', 'Brooklyn, New York City, New York, USA')]), Element(1, 'FAMS', '@F1@')])
```

### Individuals
Cannot access individuals as a whole:

```python
>>> print gedfile.individuals
<generator object <genexpr> at 0x103ef25a0>
>>> print gedfile.individual # Probably Don't.
# Don't do this. It prints all individual records in the file.
```

To access individual records:

```python
>>> for person in gedfile.individuals:
...     print person
Individual(0, 'INDI', '@P1@', [Birth(1, 'BIRT', [Element(2, 'DATE', '03 dec 1970')]), Element(1, 'SEX', 'M'), Element(1, 'NAME', 'John /Smith/'), Element(1, 'FAMC', '@F1@')])
Individual(0, 'INDI', '@P2@', [Element(1, 'NAME', 'Jane /Doe/'), Element(1, 'SEX', 'M'), Birth(1, 'BIRT', [Element(2, 'DATE', '06 nov 1946'), Element(2, 'PLAC', 'Brooklyn, New York City, New York, USA')]), Element(1, 'FAMS', '@F1@')])
```
To access individual records of a specific type use dot notation:

```python
>>> for person in gedfile.individuals:
...     print person.birth
Birth(1, 'BIRT', [Element(2, 'DATE', '03 dec 1970')])
Birth(1, 'BIRT', [Element(2, 'DATE', '06 nov 1946'), Element(2, 'PLAC', 'Brooklyn, New York City, New York, USA')])
```

To specify individual record types:

```python
>>> for person in gedfile.individuals:
...     print person.birth.date
03 dec 1970
06 nov 1946
>>> for person in gedfile.individuals:
...     print person.birth.place
# AttributeError: 'NoneType' object has no attribute 'value'
# this does not print: Brooklyn, New York City, New York, USA
```
The AttributeError is thrown when a record of that type does not exist, and by default will NOT pass onto the next record.

To bypass this error use a try/except case to pass over records that don't exist

```python
>>> for person in gedfile.individuals:
...     try:
...         print person.birth.place
...     except AttributeError:
...        print "There is no birth place record for this person"
There is no birth place record for this person
Brooklyn, New York City, New York, USA
```
##### current available use cases
```
person.birth              # birth info
person.birth.place        # string
person.birth.date         # string
person.death              # death info
person.death.place        # string
person.death.date         # string
person.name               # return first and last name as a tuple
person.father             # father full info
person.mother             # mother full info
person.parents            # list both parents full info index[0] is father index[1] is mother
person.aka                # list 'also known as' name
person.gender             # string : 'M' or 'F'
person.sex                # same as gender
person.id                 # @P12@
person.is_female          # boolean
person.is_male            # boolean
person.note               # string
person.title              # string
person.default_tag        # string tagname : 'INDI', 'FAM', etc
person.tag                # string tagname : 'INDI', 'FAM', etc

```

#### Advanced usage

Get the name of a person and parents of that person:

```python
>>> for person in gedfile.individuals:
...     try:
...         print person.name, person.parents[0].name, person.parents[1].name
...     except IndexError:
...         print "no parent name record for this person"
# OR
>>> for person in gedfile.individual:
...     try:
...         print person.name, person.father.name, person.mother.name
...     except AttributeError:
...        print "no parent name record for this person"
# either one will print:
('John', 'Doe') ('Jack', 'Doe') ('Jane', 'Doe')
('Jenny', 'Doe') ('Jack', 'Doe') ('Jane', 'Doe')
```

### Families

Family records are accessed the same way as individuals

```python
>>> print gedfile.families
<generator object <genexpr> at 0x10523c7d0>
>>> print gedfile.family # Probably don't.
# Don't do this. Prints all family records in the family
```

```python
>>> for family in gedfile.families:
...     print family
Family(0, 'FAM', '@F1@', [Husband(1, 'HUSB', '@P5@'), Wife(1, 'WIFE', '@P1@'), Element(1, 'CHIL', '@P2@', [Element(2, '_FREL', 'Natural'), Element(2, '_MREL', 'Natural')])])

>>> for family in gedfile.families:
...     print family.partners
[Husband(1, 'HUSB', '@P5@'), Wife(1, 'WIFE', '@P1@')]
```

Use cases for partners:

```python
>>> for family in gedfile.families:
...     print family.partners[0]
...     print family.partners[1]
Husband(1, 'HUSB', '@P5@')
Wife(1, 'WIFE', '@P1@')

>>> for family in gedfile.families:
...     print family.partners[0].tag
HUSB

>>> for family in gedfile.families:
...     print family.partners[0].value
@P5@
```

##### current available use cases

```
family.id           # string '@F49@'
family.tag          # string 'FAM'
family.partners     # list 
```

### dates
Dates are user input and can vary wildly in formatting. There are also approximate dates that cannot be formatted. 
These approximate dates can be stripped out using `re` or just `str.replace()`

Using pythons <a href='https://docs.python.org/2/library/datetime.html'>`datetime`</a> library (specifically <a href='https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior'>`strftime`</a> & <a href='https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior'>`strptime`</a>. the dates available can be formatted by looping through various date formats using try/except.

```python
>>> dateFormats = ['%m/%d/%Y', '%m-%d-%Y', '%d-%m-%Y', '%d %b %Y'] #just a few examples
>>> for person in filename.individuals:
...     for i in dateFormat:
...         try:
...             print datetime.strptime(person.birth.date, i)
...         except ValueError: # ValueError will be thrown when the date given does not match the formatting provided from the dateFormat list
...             pass
```
To discover more dates add a counter and increment as it passes through the dateFormat list. If the counter is higher than the length of the list -1 raise an exception printing the date that broke the program.
