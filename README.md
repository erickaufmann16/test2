Release notes for conflict parser 1.0.0

This conflict parser consumes a file of transactions in MySQL flavor like those in demo.txt and outputs the possible conflicts as in Example 15 
for further analysis. It depends on the 'regex' package, please insert the following line to the 'dependencies' section in your Cargo.toml file

regex = "1.5"

The syntax to run it in a rust environment is:

cargo run [option] input.txt

where input.txt is a file like demo.txt

supported options are:

t	Instead of regular output, prints the parsed transactions for review of syntax errors

v	Prints the version of the program

Since the conflict parser is just a helper program that aims to relieve us from tedious work of analyzing possible conflicts in a SQL application,
I didn't use a state machine for the parser. Instead, I've used simple regular expressions for the job. This implies the following rules for the input
file syntax:

1. The syntax is of MySQL flavor. Other SQL dialects might need to make appropriate adjustments before running the program

2. Names - column names, table names, etc. - can only consist of the following characters: _a-zA-Z0-9#@$

3. A variable starts with a colon symbol ':' 

4. A transaction starts with 'start transaction' which is followed by the name of the transaction like 'start transaction new_order;'

5. An update can only have ONE updated field. An update with multiple fields must be split into multiple statements

6. If the value for an update depends on a read of the updated field, the read must happen right after the assignment symbol '=', 
like in 'update ... set col =col +1;'. Syntax like 'update ... set col =1 + col;' is not allowed

7. If 6 can't be arranged, the update should be re-written as two SQL statements: one that selects the field to be updated into a variable 
and a second one that uses the variable in the expression for the field to be updated

8. A line of the form 'select_set:list', where 'list' is a comma separated list of selected fields like 'col1, col2, col3 ...'(could be empty), must be 
inserted before a select statement like 'select COUNT(*) from ...' to indicate the set of fields read is empty when 'list' is empty; or it must be 
inserted before a select statement like 'select sum(col) from ...' to supply the selected set of the statement('list' consists of one single field col 
in this specific case)to the program. This way I don't have to parse the COUNT function and such in the code. There are examples like this in 
the demo.txt file

9. The where clause is supposed to be a list of clauses separated by logical operators: &&, and, or, ||, xor. Usually a clause starts with a 
column, which is followed by operators: =, >, <, >=, <=, <>, between, like, in, and then the bounds for the column. If it is complex and 
failed to demonstrate this common form, like in '... where col1 ^ 2 + col2 ^ 2 <10000', we must insert before the statement with a line 
of the form 'd_set:list' to provide the decision set, where list is again a comma separated list of fields. For the previous specific example, it 
should be

d_set: col1, col2

Notice when a statement doesn't contain a where clause, you don't need to supply an empty d_set list. This situation is taken care by the 
program.
