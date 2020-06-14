# 账户gm3denbwguge动作分析
## transfer异常动作
{"account_action_seq": 359, "block_time": "2018-06-25T15:27:17.500", "action_trace": {"account_ram_deltas": [], "context_free": false, "console": "", "inline_traces": [], "producer_block_id": "002776ea9b151169f365ff509bdf85817a75d5710d2fb20e45530106cac04b8c", "receipt": {"act_digest": "edd275da8505813ca8e3b46e5e366bec896e2ad2c1dc8a41a3d498cce4b65aac", "recv_sequence": 118, "code_sequence": 3, "receiver": "gm3denbwguge", "global_sequence": 6927144, "abi_sequence": 3, "auth_sequence": [["gyztomjugage", 101550]]}, "except": null, "elapsed": 5, "block_time": "2018-06-25T15:27:17.500", "trx_id": "9163a1b218134050a885406ae512e445825b19965456e07b7c91623d45cae782", "act": {"account": "gyztomjugage", "data": "a09861fa499abf67a09866fc4c95866460127f5900000000044345544f530000136365746f7320746f6b656e2061697264726f70", "name": "transfer", "authorization": [{"actor": "gyztomjugage", "permission": "active"}]}, "block_num": 2586346}, "global_action_seq": 6927144, "block_num": 2586346}

gyztomjugage::代币转账gyztomjugage @active
gyztomjugage -> gm3denbwguge
150150.0000 CETOS (:gyztomjugage)
(MEMO: cetos token airdrop)
这个是CETOS币，数据部分直接就是一串字符串来表示交易的内容，这种币暂时不给于解释

## 非transfer动作
receipt_receiver = "gyztomjugage",一次转账操作会通知3方，通过检验票据的接收者可以来去掉重复的transfer交易

claim  
event  
playgame  
ping  
drawreceipt  
msg  
joinus  
info  
notice  
signup  
luckreceipt  
alert  
broadcast  
hi  
eosabcd  
news  
dt  
promote  
topnews  
a  
lotreceipt  
paytxfee  
i  
m  
o  
n  
news1113  
u  
push  
yell


receipt_receiver ！= "gyztomjugage" 一些操作可能不会通知"gyztomjugage",也就是对该账户产生票据

bidname = 0280d4d842481acbfe69f0b8e37baad87785c8dffdc9bbf7c48a0c2ae3c2b344
buyram = b1adef232f0718805e522a5b36600f754989bd423ba169eb310f4fbc30f4ea80
buyrambytes = 7382c6b300cdabf7840e1129498779187983e48b5ae036342bf065567b7888e3
claim = 30f1a29807ed9e8cf23f840047aaf2fa28f8a798e6618c7a81eaac2238206189
delegatebw = f17c33a1aeff4cf338871cdd60cfc3e2a75fae46812da98a6fda99634a48c5d4
draw = 6ecf1b9e04ff0e234d6e6e11ccc596ec3af4db19b0c0e1778b2d0553a8fe5f02
login = 1b9ec22016597fdf25a33a76034046870bb13cec5296ec7da39b0b2b9e838716
luck = 994bee15373b2339277792f3ef0c8f43ec54f34258f9ad21dc6fd4f582038dd5
memberreg = bc139fb10421c256d29706b2a69a4f73f4470588d854a50bc5ca6db2e1f9dee4
newaccount = 7382c6b300cdabf7840e1129498779187983e48b5ae036342bf065567b7888e3
open = 9179fbb888d2b300d0f6a685a8e08bb0e9e0e1e4983e8ae618c5e1075d03352d
paytxfee = f63e0f7622f1c8ab5adbb1d1c9f2bd650e0c8fe6c53fa911268c1e5481223d3c
refund = c165e7c0440f85f0eac7c0371f9e4e31503ec55419a4628b1502b2bacc90bae5
signup = 03e33e5e9e07f0106f2ba290a2ac5c29f7eb0f66b9d7d1b98ca92626410970e8
undelegatebw = fce71c9a5c6420de75a9aa89fcb7f9474e7bbbe6e9e4230f8f870f6109ea92ce
voteproducer = 1a0a7ad28b8dc06957de07872c60df9075913076a77a86b795af38953b094829

bidname  0280d4d842481acbfe69f0b8e37baad87785c8dffdc9bbf7c48a0c2ae3c2b344
eosio::竞拍账户gm3denbwguge @active
gm3denbwguge 以 6051.1100 EOS 的价格竞拍账号 org
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.names
6051.1100 EOS (:eosio.token)
(MEMO: bid name org)

buyram  7c0b87c888ee313472495e83f1f1060b6b0210a2136e6c9c275c78f1c5cdb006
eosio::购买内存gm3denbwguge @active
gm3denbwguge 为 prochaintest 购买了价值 2.0000 EOS 的内存
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ram
1.9900 EOS (:eosio.token)
(MEMO: buy ram)
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ramfee
0.0100 EOS (:eosio.token)
(MEMO: ram fee)

buyrambytes   9215795c1af5547f1e407422a157d0d7b8cddf2a2e3e724835f6af95fc5841f7
eosio::购买内存gm3denbwguge @active
gm3denbwguge 为 ex 购买了 8192 bytes内存
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ram
0.9494 EOS (:eosio.token)
(MEMO: buy ram)
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ramfee
0.0048 EOS (:eosio.token)
(MEMO: ram fee)
eosio::抵押资源gm3denbwguge @active
gm3denbwguge 为 ex 抵押了价值 0.1000 EOS 的带宽和价值 10.0000 EOS 的CPU，未转移EOS
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.stake
10.1000 EOS (:eosio.token)
(MEMO: stake bandwidth)
eosio::创建账户gm3denbwguge @active
gm3denbwguge 创建了账户 ex

claim  30f1a29807ed9e8cf23f840047aaf2fa28f8a798e6618c7a81eaac2238206189
hirevibeshvt::claimgm3denbwguge @active
{
  "owner": "gm3denbwguge",
  "sym": "4,HVT"
}

delegatebw  a4a3d0628a2664af30cce77ac8eafdb963dfdd65c4e64d53a99d5d34815f8299
eosio::抵押资源gm3denbwguge @active
gm3denbwguge 为 candy.pra 抵押了价值 20.0000 EOS 的带宽和价值 2000.0000 EOS 的CPU，未转移EOS
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.stake
2020.0000 EOS (:eosio.token)
(MEMO: stake bandwidth)

draw  105cbce7a2d58b4584c40d1a913124df8b90b2fa8480fb89ad2a980f2f0fee53
betdicelucky::drawgm3denbwguge @owner
{
  "from": "gm3denbwguge"
}

login  1b9ec22016597fdf25a33a76034046870bb13cec5296ec7da39b0b2b9e838716
roulettespin::logingm3denbwguge @owner
{
  "account": "gm3denbwguge",
  "ref": "zealotzealot"
}

luck 994bee15373b2339277792f3ef0c8f43ec54f34258f9ad21dc6fd4f582038dd5
eosluckydice::luckgm3denbwguge @active
{
  "actor": "gm3denbwguge",
  "sub": 1464217361
}  

memberreg  bc139fb10421c256d29706b2a69a4f73f4470588d854a50bc5ca6db2e1f9dee4
eosdactokens::memberreggm3denbwguge @active
{
  "sender": "gm3denbwguge",
  "agreedterms": "6d2cc6201302b3f485e2a939881ae451"
}

newaccount d5d6614c0a57825fb9fb65b2dfa2d3b68273adfbecc9a7350e3190932a1152e5 
eosio::购买内存gm3denbwguge @active
gm3denbwguge 为 prochainfree 购买了 8192 bytes内存
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ram
2.5539 EOS (:eosio.token)
(MEMO: buy ram)
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.ramfee
0.0129 EOS (:eosio.token)
(MEMO: ram fee)
eosio::抵押资源gm3denbwguge @active
gm3denbwguge 为 prochainfree 抵押了价值 1.0000 EOS 的带宽和价值 10.0000 EOS 的CPU，未转移EOS
eosio.token::代币转账gm3denbwguge @active
gm3denbwguge -> eosio.stake
11.0000 EOS (:eosio.token)
(MEMO: stake bandwidth)
eosio::创建账户gm3denbwguge @active
gm3denbwguge 创建了账户 prochainfree



### 注意
newaccount = d5d6614c0a57825fb9fb65b2dfa2d3b68273adfbecc9a7350e3190932a1152e5
***************
receipt_receiver = eosio
buyrambytes = d5d6614c0a57825fb9fb65b2dfa2d3b68273adfbecc9a7350e3190932a1152e5
***************
receipt_receiver = eosio
delegatebw = d5d6614c0a57825fb9fb65b2dfa2d3b68273adfbecc9a7350e3190932a1152e5
这3个动作有同一个交易id，到时候注意查看原始数据时进行区分

open  9179fbb888d2b300d0f6a685a8e08bb0e9e0e1e4983e8ae618c5e1075d03352d
infinicoinio::opengm3denbwguge @owner
{
  "owner": "gm3denbwguge",
  "symbol": "4,INF",
  "ram_payer": "gm3denbwguge"
} 
paytxfee 645db58ae3c821743bd1314d34b9151222677a767c9d0e2f790e0a70031d2f9a
everipediaiq::代币转账gm3denbwguge @active
gm3denbwguge -> newdexpocket
130000.000 IQ (:everipediaiq)
(MEMO: {"type":"sell-limit","symbol":"IQ_EOS","price":"0.00220","count":130000,"amount":286})
everipediaiq::paytxfeegm3denbwguge @active
{
  "from": "gm3denbwguge",
  "quantity": "130.000 IQ",
  "memo": "0.1% transfer fee"
}

refund  0bfc58151b8622650d10eb9cff58f299e805e6b313ba484208e74b44dbce6f7f
eosio::refundgm3denbwguge @active
{
  "owner": "gm3denbwguge"
}
eosio.token::代币转账eosio.stake @active
eosio.stake -> gm3denbwguge
79990.0000 EOS (:eosio.token)
(MEMO: unstake)


signup  03e33e5e9e07f0106f2ba290a2ac5c29f7eb0f66b9d7d1b98ca92626410970e8
eoscubetoken::signupgm3denbwguge @active
{
  "owner": "gm3denbwguge",
  "quantity": "0.0000 CUBE"
}

undelegatebw  91310eb08677c427bba436f515b6eddbc9b0db1ac094b00cce9b1a49c38e6a6e
eosio::赎回资源gm3denbwguge @active
gm3denbwguge 赎回了之前为 prochainfree 抵押的价值 0.0000 EOS 的带宽和价值 1500.0000 EOS 的CPU

voteproducer  1a0a7ad28b8dc06957de07872c60df9075913076a77a86b795af38953b094829
eosio::投票gm3denbwguge @active
gm3denbwguge 将票委托给了代理 eosvoteproxy

a  
message.bank::a xzleoswallet @active
{
  "u": "gm3denbwguge",
  "msg": "⛲ XPET版征战全球《X争霸》北京时间今晚8点（11月8日20点）启航，没有输家的游戏，终极大奖+bancor代币+宠物分红，每块领地都极具价值，挑战最后的征服者，带走1000eos始祖神兽。 https://www.xpet.io/xmap.html"
}

alert
eosblackdrop::alerte osblackdrop @active
{
  "account": "gm3denbwguge",
  "message": "Bonus eosBLACK tokens pending. Transaction was not signed or wrong key signature. Please try again at https://eos-black.com"
}


broadcast
betdicealert::broadcast betdicealert @active
{
  "message": "복권 시작! 22K EOS는 이미 보상을 받았습니다! 와서 다음 차례가 되어라! www.betdice.one"
}

claim  bcf86547a1b23c2728f51162a0a19d404628b3c74fa32ce514217b5ec791e935
thedeosgames::claimgm3denbwguge @active
{
  "owner": "gm3denbwguge",
  "quantity": "0.0000 DEOS"
}

drawreceipt  6536ad0b9f8cbadb38fd2a1ab39b1aaae11be8b0a1afcef27b90c8662661a491
betdicelucky::drawrevealbetdicelucky @active
{
  "name": "gm3denbwguge",
  "signature": "SIG_K1_Kc8MbWZUnx9draDC3TaGrNz8xABEmjpqGZh4svXajKxnd4LsCrYrynGtUQ4q8cKKvUXTgNxauk8aUESEPSyB4qigMXRwcU"
}
eosio.token::代币转账betdicelucky @active
betdicelucky -> gm3denbwguge
0.0005 EOS (:eosio.token)
(MEMO: Welcome back to BetDice( https://betdice.one )! Here is your reward of Lucky Draw!)
betdicelucky::drawreceiptbetdicelucky @active
{
  "name": "gm3denbwguge",
  "seed": "6e3e9653e17136a4a6d9cc851a3af6c5fcc8086f8c8b0e404f4d39c5f5e0cb38",
  "signature": "SIG_K1_Kc8MbWZUnx9draDC3TaGrNz8xABEmjpqGZh4svXajKxnd4LsCrYrynGtUQ4q8cKKvUXTgNxauk8aUESEPSyB4qigMXRwcU",
  "luckyNumber": 3012,
  "winAsset": "0.0005 EOS",
  "drawTime": 1538583064
}

dt    7fda4a7ebff4305ae7a9a2568cc3a31333b5dd65911ae926dd0a56ea2747a989
message.bank::dtnvzhuangyuan @active
{"u":"gm3denbwguge","msg":"?The XPET version of the WORLD CONQUEST "X hegemony" is about to open, there is no loser game, the ultimate award + bancor token + pet dividends, each territory is extremely valuable, challenge the final conqueror, take away the 1000eos ancestor beast. https://www.xpet.io/xmap.html"}

eosabcd 278dc4c03ccd8f30a94e748286fdf0ab18eefa23f100717b3e1107cce870bba2
eoseosaddddd::eosabcdgm2tkmbyguge @active
{
  "from": "gm2tkmbyguge",
  "to": "gm3denbwguge",
  "memo": "承接区块链空投广告，用于dapp·公众号·平台等推广。全网最低价，1万条只要2个eos，欢迎骚扰，微信：eosabcd"
}

event  2b85b3917a53f2ef997b28f3656dcabf8de4bd8e143cc2322ecc12a1e6203dbf
rockscissors::eventgameboyadmin @active
{
  "winner": "gm3denbwguge",
  "memo": "【 https://eosgameboy.io/ 】 • RPS game is the latest RETRO style game dapp. During launching event period, You can get 4x more GAMEBOY token! • 짱껜뽀 게임은 복고풍 스타일 게임 dapp입니다. 게임보이 토큰은 게임런칭행사기간에만 최대 4배 더 드랍되므로 지금 접속해서 플레이하세요. • 新Dapp游戏到了!!石头剪子布石就可以获得5倍EOS令牌. 另外，我们赠送巨额BOYtoken奖金有限的动期。 【 https://t.me/eosgameboy_dapp 】"
}

hi  d654750ebd65a71441cdf95c321371997380d90a12477310aa606858bf9c41d6
1hello1world::hi1hello1world @active
{
  "recipient": "gm3denbwguge",
  "memo": "最私密的EOS账户注册方法，支付宝、微信支付购买EOS账号。https://meet.one/invitecode/buy"
}

i  7cb4e0cea106cc36a0135ecdeb321f9734cc43678e77d356b111a6a8d898ebbc
contractbase::ismallaccount @active
{
  "u": "gm3denbwguge",
  "msg": "✈ XPET版征战全球《X争霸》已开启，没有输家的游戏，终极大奖+bancor代币+宠物分红，每块领地都极具价值，挑战最后的征服者，带走1000eos始祖神兽。 https://www.xpet.io/xmap.html"
}

info  6799912e06505c7722f507a8abb0715dce5b5741ac89e224f37b4cbf3d8e549b
contractbase::infosmallaccount @active
{
  "u": "gm3denbwguge",
  "msg": "⏬ The XPET version of the WORLD CONQUEST \"X hegemony\" was already live, there is no loser game, the ultimate award + bancor token + pet dividends, each territory is extremely valuable, challenge the final conqueror, take away the 1000eos ancestor beast.  https://www.xpet.io/xmap.html"
}
contractbase::infosmallaccount @active
{"u":"gm3denbwguge","msg":"⏬ The XPET version of the WORLD CONQUEST "X hegemony" was already live, there is no loser game, the ultimate award + bancor token + pet dividends, each territory is extremely valuable, challenge the final conqueror, take away the 1000eos ancestor beast.  https://www.xpet.io/xmap.html"}

joinus  0e83f02d8cd290b58ff9e34f6f583aad60dfd76019348e7ca743cba0d0f9a54d
rockscissors::joinusgameboyadmin @active
{
  "player": "gm3denbwguge",
  "memo": "【https://eosgameboy.io/ 】 • RPS game is the latest RETRO style game dapp. During launching event period, You can get 4x more GAMEBOY token! • 짱껜뽀 게임은 복고풍 스타일 게임 dapp입니다. 게임보이 토큰은 게임런칭행사기간에만 최대 4배 더 드랍되므로 지금 접속해서 플레이하세요. • 新Dapp游戏到了!!石头剪子布石就可以获得5倍EOS令牌. 另外，我们赠送巨额BOYtoken奖金有限的动期。 【 https://t.me/eosgameboy_dapp 】"
}

lotreceipt  

luckreceipt  98de823c85a4082ee7a2a3ef3219b73632d23f5e1170721cf9024bb240fa87f3
eosluckydice::luckreceipteosluckydice @active
{
  "name": "gm3denbwguge",
  "seed": "015ad09b4a748ccb1e0857602b2460de27e74af9044530e468aa171bf4c46f6d",
  "roll_value": 4789,
  "reward": "0.0006 EOS",
  "draw_time": "1539150367500000"
}
eosio.token::代币转账eosluckydice @active
eosluckydice -> gm3denbwguge
0.0006 EOS (:eosio.token)
(MEMO: EOS.Win lucky draw rewards!)

m  
msg 消息 

n  3b9188091db7d5adbd12100938d83c55df12446f31ec33d71c0959f3c0493b37
message.bank::nsmallaccount @active
{
  "u": "gm3denbwguge",
  "msg": "基于WebGL的《征战世界》升级版，酷炫的3D效果，让资金盘不再单调。http://cryptomeetup.io 。 路线图: https://github.com/crypto-meetup-dev/cryptomeetup-portal/issues/1"
}

news 9e17c0f2d7aafcbc9b5a0a6c24828e771dcac493d85a6546b6983ef42a0120d8
message.bank::newsyourman.bank @active
{
  "u": "gm3denbwguge",
  "msg": "New world conquest game with Group buys, Continent battles and Mini games inside every country, never ending, Just Fight!  https://playeos.co "
}

news1113  
notice  
o  
paytxfee  
ping  ecc08dc6806edf10c970560f9bdad36a6d42135939c3bf3b5ad63636217aff05
watchdoggiee::pingmesscomposer @active
{
  "from": "messcomposer",
  "to": "gm3denbwguge",
  "memo": "EOSBET - Provably fair EOS Dice Casino! - Play now for huge profits, use the following referal link for a 0.5% win bonus! https://dice.eosbet.io/?ref=messcomposer"
}

playgame  364a00d45b642b416baf4e94fde8cf2407632d390f53a325fa112be342f343ad
gameboylucky::playgamegameboylucky @active
{
  "player": "gm3denbwguge",
  "memo": "Gameboy 收益共享赌博平台已经上线，公平可验证的EOS游戏玩法！获得4倍代币以在有限的时间内获得利润分成. https://eosgameboy.io"
}

promote  f401b3fc4f6503d3b19a562ba76f2be21c9ef7e08e02d0e84182eb63ba8c9cbb
eospromoter1::promotemesscomposer @active
{
  "recipient": "gm3denbwguge",
  "memo": "10 MILLIONTH bet promo at EOSBet Dice (dice.eosbet.io)! Get one BET token for every 5 EOS bet, and get dividends for life!"
}

push 消息

signup  632fd2d63db5f76275377c8e68d8d9cf8b7fca59c091979910ffe695f15737ad
eoscubetoken::signupgm3denbwguge @active
{
  "owner": "gm3denbwguge",
  "quantity": "0.0000 CUBE"
}

topnews  消息
u  c13a5b2f2dbe47d0ec8b7df2a749694b1660d4eae9220c7adc6c4d609b2d40b4 消息
message.bank::usmallaccount @active
{
  "u": "gm3denbwguge",
  "msg": "⤵ The XPET version of the WORLD CONQUEST \"X hegemony\" is about to open, there is no loser game, the ultimate award + bancor token + pet dividends, each territory is extremely valuable, challenge the final conqueror, take away the 1000eos ancestor beast.  https://www.xpet.io/xmap.html"
}
message.bank::usmallaccount @active
{"u":"gm3denbwguge","msg":"⤵ The XPET version of the WORLD CONQUEST "X hegemony" is about to open, there is no loser game, the ultimate award + bancor token + pet dividends, each territory is extremely valuable, challenge the final conqueror, take away the 1000eos ancestor beast.  https://www.xpet.io/xmap.html"}

yell 7f8fdb046a943444bd4aadabee3b9d38899e2f045c19600503f10f17f862dba1
eosplayaloud::yelleosplayaloud @active
{
  "u": "gm3denbwguge",
  "memo": "EOSPlay Lottery & Dice, FREE EOS Giveaway! https://eosplay.co/link/eosplayaloud?11734dc5"
}



# cleos 命令
cleos -u https://nodes.get-scatter.com:443 get actions gm3denbwguge  1 100