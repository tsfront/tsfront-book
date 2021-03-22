# 计时器调度

> 前端开发过程中经常遇到设置计时器，比如在登陆框下方的发送验证码，会设置60秒的计时器。之所以这么设计，一方面是在于用户体验、让用户对于等待多久有预期（类似缓冲时候的缓冲进度、转圈圈功能），另一方面也是阻止恶意频繁获取验证码（所以需要在计时器启动后，计时器按钮置灰、不可点击），以免对后端服务造成不必要的压力。
>
> 以下是在开发获取验证码计时器的经验，总结：
>
> 1. 【失败】组件内新建handler函数操作`setInterval`，并在「获取验证码」按钮`onClick`事件中调用
>
> - 使用`setCounter(counter => counter -1)`可以对`counter`这个状态值进行改变。⚠️但`setCounter(counter -1)`是不行的！！！与下面第二条本质是一个道理。
> - 无法进行计时器终止条件的判断，即无法在handler函数中获取到`counter`最新状态，也就无法执行`if (counter === 0)`判断。
>
> 2. 【失败】在useEffect中设置`setInterval`，因为需要手动触发（点击「获取验证码」按钮）计时器，需要`useEffect(()=>{setCounter(counter => counter -1)}, [counter])`第二参数依赖于`counter`。⚠️ 但这样会创建多个interval计时器，造成`counter`状态变化异常跳动。并且无法终止计时器，一直到负无穷。

方法1：使用`setInterval`且利用clean-up函数及时清除上一个已创建的interval计时器。

```jsx
const Clock = () => {
  const [counter, setCounter] = useState(0); //初始值为0
  const [btnDisable, setBtnDisable] = useState(false); //指示「获取验证码」按钮可用/置灰变量
  
  // ⚠️ useEffect执行条件
  useEffect(() => {
    // 计时器开始后，设置按钮disable
    setBtnDisable(true);
    
    let id = setInterval(() => {
      // 这里不能使用 setCounter(c-1)
      setCounter(c => c-1);
      // 每隔1秒执行一次的计时器
    }, 1000);
    
    // ⚠️ 这里判断条件有两个作用
    // 1.组件刚挂载完成后执行effect，无需开始计时。
    // 2.经用户手动触发计时到0时刻的时候，需要能停止最后一个计时器（阻止它继续变动counter值）。
    if (counter === 0) {
      clearInterval(id);
      setBtnDisable(false);
    }
    // clean-up函数每次清楚上一个已存在的计时器，避免多个计时器并存、混乱修改counter值
    return () => clearInterval(id)
  }, [counter])
  
  return (
    <div>
      <button disable={btnDisable} onClick={() => setCounter(60)} >
        获取验证码
      </button>
    </div>
  
  )
}
```

方法2：上一个例子看出，区间计时器其实只执行了一轮，就被clear，很符合`setTimeout`特性，所以考虑使用后者。唯一差异点，省去了clean-up函数。

```jsx
const Clock = () => {
  const [counter, setCounter] = useState(0);
  const [btnDisable, setBtnDisable] = useState(false); 
  
  useEffect(() => {
    setBtnDisable(true);
    
    let id = setTimeout(() => {
      setCounter(c => c-1);
    }, 1000);
    
    if (counter === 0) {
      // ⚠️ 这个位置 clearTimeout 不能少
      clearTimeout(id);
      setBtnDisable(false);
    }
  }, [counter])
  
  return (
    <div>
      <button disable={btnDisable} onClick={() => setCounter(60)} >
        获取验证码
      </button>
    </div>
  
  )
}
```

