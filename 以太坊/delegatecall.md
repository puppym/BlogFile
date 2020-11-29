> 本文主要介绍call, callcode, delegatecall三者之间的区别
>
> 总结下来主要有两个方面的区别：1. 上下文。2. msg.sender。对于call而言一次call的调用改变了合约的上下文环境，而对于callcode和delegatecall没有改变上下问的环境。对于call,callcode的msg.sender都是调用栈当前调用者的地址，delegatecall的msg.sender是调用栈最初的地址。tx.orign为交易的发起者。

delegatecall简单来说就是“委托”被调用者去管理自己的上下文环境(storage)。

delegatecall是修复了callcode没有保留调用栈最初的msg.sender以及msg.value信息的bug。如果A调用B，然后BdelegatecallC，在delegatecall中B调用C的msg.sender是A，如果是call，那么msg.sender是B。

```js
contract D {
  uint public n;
  address public sender;

  function callSetN(address _e, uint _n) {
    _e.call(bytes4(sha3("setN(uint256)")), _n); // E's storage is set, D is not modified 
  }

  function callcodeSetN(address _e, uint _n) {
    _e.callcode(bytes4(sha3("setN(uint256)")), _n); // D's storage is set, E is not modified 
  }

  function delegatecallSetN(address _e, uint _n) {
    _e.delegatecall(bytes4(sha3("setN(uint256)")), _n); // D's storage is set, E is not modified 
  }
}

contract E {
  uint public n;
  address public sender;

  function setN(uint _n) {
    n = _n;
    sender = msg.sender;
    // msg.sender is D if invoked by D's callcodeSetN. None of E's storage is updated
    // msg.sender is C if invoked by C.foo(). None of E's storage is updated

    // the value of "this" is D, when invoked by either D's callcodeSetN or C.foo()
  }
}

contract C {
    function foo(D _d, E _e, uint _n) {
        _d.delegatecallSetN(_e, _n);
    }
}
```

如上面的代码的调用栈为C->D->E：

* 当D call E，代码运行在合约E的上下文环境中，当前调用的setN的值为E的storage中的值，在E中msg.sender为D(被调用者的storage值可能发生变化)。
* 当D callcode E，代码运行在D的上下文环境中，就像E的代码嵌入到D的代码中运行一样，当前调用setN的值为D的storage中的值，在E中msg.sender为D(调用者的storage值可能发生变化)。
* 当D delegatecall E，代码运行在D的上下文环境中，当前setN的值为D的storage中的值，D和E的msg.sender都为C(调用者的storage值可能发生变化)。