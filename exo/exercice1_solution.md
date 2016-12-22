# Exercice 1 - Solution

##### Fraxken solution (typescript)
```ts
(async () => {

    const taskArray : Promise<any>[] = [];

    function manageLoop() : Promise<void> {
        return new Promise<void>( (resolve,reject) => {
            for(let i: number = 1;i< 5 ;i++) {
                const copy: number = i;
                process.nextTick(() => {
                    const randomTimer: number = Math.floor(Math.random() * (9000 - 1500 + 1)) + 1500;
                    taskArray.push( new Promise<any>( (resolve,reject) => setTimeout( _ => {
                        console.log(`Task ${copy} done`);
                        resolve();
                    }, randomTimer ) ) );
                });
                if(i === 4) resolve();
            }
        });
    }

    console.log('await loop');
    await manageLoop();

    console.log('await promise!');
    await Promise.all(taskArray);

    console.log('All tasks are done!');

})();
```

##### Purexo and Vababallz solution 

```js
const taskArray = []
const ordertaskResolve = []
const nextTicks = []

for (let i = 1; i < 5; i++) {
    const index = i

    nextTicks.push(new Promise((resolve, reject) => {
        process.nextTick(() => {
            const randomTimer = Math.floor(Math.random() * (9000 - 1500 + 1)) + 1500

            taskArray.push(new Promise((resolve, reject) => {
                setTimeout(_ => {
                    console.log(`Task ${index} done !`)
                    ordertaskResolve.push(index)
                    resolve()
                }, randomTimer)
            }));

            resolve()
        });
    }))
}

console.log('await promise!')
Promise.all(nextTicks)
    .then(_ => Promise.all(taskArray))
    .then(_ => console.log('All tasks are done in order : ', ordertaskResolve))
```
