# Exercices 1 - Async control 

##### Description : 

Something is not working correctly (fix the code).

##### Expected :
- Wait for task instanciation. 
- Wait for task execution (log task resolution as well).
- Log a final message.

##### ES6 Version : 

```js
( () => {
    const taskArray = [];
    var i = 1;

    for(;i< 5 ;i++) {
        process.nextTick(() => {
            const randomTimer = Math.floor(Math.random() * (9000 - 1500 + 1)) + 1500;
            taskArray.push( new Promise( (resolve,reject) => setTimeout( _ => {
                resolve();
            }, randomTimer ) ) );
        });
    }

    console.log('await promise!');
    await Promise.all(taskArray);

})();
```

##### TypeScript : 

```ts
(async () => {

    const taskArray : Promise<any>[] = [];
    var i: number = 1;

    for(;i< 5 ;i++) {
        process.nextTick(() => {
            const randomTimer: number = Math.floor(Math.random() * (9000 - 1500 + 1)) + 1500;
            taskArray.push( new Promise<any>( (resolve,reject) => setTimeout( _ => {
                resolve();
            }, randomTimer ) ) );
        });
    }

    console.log('await promise!');
    await Promise.all(taskArray);

    console.log('All tasks are done!');

})();
```

# Constraint 

- Only NodeJS packages are allowed.
- You can use TypeScript,ES6 or Babel (like you want).
- --harmony flag is allowed.

# Solution terminal output 

Message can be edited by yourself for something that match your solution !

```
Await loop
Tasks processed!
Task 4 done
Task 2 done
Task 3 done
Task 1 done
Tasks are done
```
