# getHashCodeFromInvoice
This is a query that retrieves all invoice data from the Sap B1 database and validates the hash code.

There was a problem where notes were being generated with the wrong Hash Code field, and to check which notes were with this problem, I developed a query in SQL where I performed this verification and pointed out which notes really had problems in generating the hash code.

Query starts by taking the fields that will be used in the concatenation to mount the hash manually, including the hashcode that is in the system that is supposed to be wrong.
Then the concatenation function is used within a Hashbytes function for the generation of the MD5 hash, which is the format used by the city in the generation of the hash. After concatenating the query makes the comparison of the concatenated fields that formed the hash with the field that is with the current hash, if there is a divergence the query will return which notes have a problem in the hashcode.
