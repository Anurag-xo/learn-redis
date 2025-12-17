## Transactions

#### Transactions allow the execution of a group of commands in a single step with 2 important guarantees..

1. All the commands are serialized and executed sequentally
2. Either ALL or NONE of the commands are executed

#### A Transaction is entered using the `MULTI` command. At that point the user can issue commands and they will not be executed but put into a queue until **EXEC** is called.
