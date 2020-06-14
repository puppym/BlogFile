# EOS权限问题
[对应官网的例子](https://developers.eos.io/eosio-home/docs/sending-an-inline-transaction-to-external-contract)
require_auth(get_self()); 
* get_self()(Get this contract name)能够返回当前的合约名称，一般合约名称就是创建的加载合约的账户，一些合约中的action只有合约的创建者能够调用，需要在这些action的前面加上这些校验操作。该操作因该调用了require_recipient(合约账户)，将合约账户添加到需要被通知的账户集集合中。官网上的例子Now if user bob calls this function directly, but passes the parameter alice the action will throw an exception。bob应该是合约的拥有者。
'''bash
  [[eosio::action]]
  void notify(name user, std::string msg) {
    require_auth(get_self());
    require_recipient(user);
  }
'''

require_recipient(user);
* 添加需要被通知的账户接收票据，因为例如一个转账操作会通知合约方，转账方，接收方。通过票据的接收者可以区分这3中情况。

require_auth( user );
* addressbook <= addressbook::upsert          {"user":"alice","first_name":"alice","last_name":"liddell","street":"1 coming down","city":"normalla...

void notify(name user, std::string msg)
require_auth(get_self());
* addressbook <= addressbook::notify          {"user":"alice","msg":"alice successfully emplaced record to addressbook"}
Notified

require_recipient(user);
* alice <= addressbook::notify          {"user":"alice","msg":"alice successfully emplaced record to addressbook"}

require_auth( name("addressbook"));
* abcounter <= abcounter::count             {"user":"alice","type":"emplace"}
warning: transaction executed locally, but may not be confirmed by the network yet    ]

# EOS 内联动作
内联动作可以有合约内的内联动作和合约间的内联动作。下面给出官网的例子
'''c

'''