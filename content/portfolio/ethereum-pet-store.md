---
title: "Cửa hàng thú cưng trên Ethereum blockchain - Phần 1"
date: 2017-10-01T15:45:42+07:00
draft: false
image: "img/portfolio/petshop.png"
---

Theo hiểu biết hạn hẹp của mình thì [Truffle](http://truffleframework.com/) mà một trong những nền tảng dễ sử dụng để phát triển các ứng dụng 
[Ethereum Smart Contract](http://truffleframework.com/tutorials/ethereum-overview). 
Bài viết này của mình chỉ mang tính tóm tắt lại các bước để làm ra một ứng dụng smart contract
theo tutor của [Truffle Pet Store](http://truffleframework.com/tutorials/pet-shop) và có bổ sung thêm một vài giải thích nhỏ của riêng mình.
Thực hiện thực hành nhanh theo tutor thì cũng nhanh thôi, chỉ tầm 1-2h đồng hồ là cùng. 
Tuy nhiên, viết lại các bước mình đã làm cũng là một cách tìm hiểu kỹ hơn cho cá nhân mình.

Nội dung chính bao gồm 3 phần:

* Cài đặt môi trường phát triển
* Tạo một ứng dụng Truffle từ mẫu có trước được lấy từ `Truffle Box`
* Viết và chạy code test cho smart contract trên local private blockchain network

## Tóm tắt về dự án

Ông chủ cửa hàng thú cưng muốn áp dụng [Ethereum](https://ethereum.org) để tăng tính hiệu quả trong việc mua bán thú cưng cho các "cậu ấm cô chiêu."
Do cửa hàng còn bé nên chỉ đủ chỗ cho tối đa 16 con thú cưng các loại thôi. 
Ông chủ yêu công nghệ này muốn gắn kết mỗi giao dịch mua bán thú cưng với một địa chỉ cụ thể của ehtereum blockchain. 
Giao diện đã được ông chủ thuê một nhà thiết kế UI/UX lững lẫy thế giới làm sẵn rồi, công việc của chúng ta chỉ đơn giản là phát triển một smart contract và viết thêm xử lý logic ở phía front-end.

Mọi thứ đã sẵn, chúng ta bắt tay vào phát triển ứng dụng theo yêu cầu của ông chủ cửa hàng yêu thích công nghệ. 
Đầu tiên thì chúng ta cần cài đặt môi trường phát triển.

## Cài đặt môi trường phát triển

Để cài đặt TestRPC và Trufle thì trước tiên chúng ta cần có vài thứ sau đây; chắc máy cuder nào cũng có sẵn cả rồi.

* Node.js v6+ và npm
* Git

Sau đó, chúng ta tiến hành cài đặt TestRPC và Truffle.

```
npm install -g ethereumjs-testrpc
npm install -g truffle
```

Tiện đây thì giải thích thêm một chút về *TestRPC* không các bạn lại thắc mắc không biết *TestRPC* là cái củ đậu gì? 
*TestRPC* đơn giản chỉ là một blockchain network chạy trên máy local của chúng ta; thường được sử dụng để chạy test cho smart contract và các DApp trên local.
Ngoài ra, chúng ra có thể deploy trên các public network như [Ropsten](http://pool.ropsten.ethereum.org), [Kovan](https://kovan.etherscan.io) hay thậm trí là trên testnet của Enthereum luôn.

Công việc cài đặt như vậy đã hòm hòm, phần thú vị nhất bây giờ là viết một smart contract theo mong muốn của ông tỷ phú tiền điện tử.

## Tạo một Truffle project

Như đã đề cập ở trên, Truffle có một số dự án mẫu nên chúng ta có thể dễ dàng khởi tạo một dự án mới dựa trên những dự án mẫu đã có sẵn bằng cách sử dụng `truffle unbox`

```
// Create the directory.
mkdir pet-shop-tutorial

// Navigate to within the directory.
cd pet-shop-tutorial

// Initialize Truffle with the base pet shop code
truffle unbox pet-shop
```

> Ngoài ra chúng ra có thể khởi tạo một dự án Truffle rỗng bằng cách sử dụng lệnh `truffle init bare`. 
> Không liên quan lắm nhưng từ `bare` khiến tôi nhớ lại thời đọc vài cuốn tiểu thuyết của Anh-Mỹ khi họ mô tả về các cô gái đẹp. :)

## Cấu trúc thư mục dự án

Cấu trúc một dự án Truffle cơ bản bao gồm các thành phần sau:

* `/contracts`: chứa những file mã nguồn [Solidity](https://solidity.readthedocs.io/en/develop/) mô tả các smart contract của dự án. 
* `/migrations`: Truffle sử dụng migration để deploy các smart lên blockchain network.
* `/test`: chứa Javascript và *Solidity* code test cho các smart contract.
* `truffle.js`: configuration file của Truffle

## Định nghĩa một *Smart Contract*

Chúng ta sẽ phát triển một ứng dụng phân tán, DApp, bằng cách định nghĩa một *smart contract* đóng vai trò như là một xử lý logic nghiệp vụ và lưu trữ ở phía *back-end*.

Trước tiên chúng ta tạo một file `Adoption.sol` trong thư mục `/contracts` với nội như sau

```
pragma solidity ^0.4.4;

contract Adoption {

}
```

Có hai thứ quan trọng các bạn cần chú ý ở đoạn code trên

1. Phiên bản tối thiểu của Solidity được định nghĩa ở trên cùng `pragma solidity ^0.4.4;` 
2. Giống như hầu hết các ngôn ngữ lập trình khác, `Solidity` sử dụng dấu `;` là ký tự phân cách các dòng lệnh.

`Solidity` là một ngôn ngữ lập trình có static-type, nên chúng ta bắt buộc phải khai báo kiểu dữ liệu cho các tham biến có kiểu như `string`, `int` hay `array`.
Ngoài ra, `Solidity` có thêm một kiểu dữ liệu đặc thù là `address` độ dài *20 bytes* dùng để lưu trữ địa chỉ trên `Ethereum` blockchain.
Các account hay smart contract trên `Ethereum` blockchain đều có một địa chỉ đến có thể gửi/nhận dữ liệu đi đến những địa chỉ đó.

Để lưu trữ địa chỉ của 16 chú thú cưng trên `Ethereum` blockchain, thì chúng ra khai báo một mảng địa chỉ có 16 phần từ như sau trong smart contract.

```
address[16] public adopters;
``` 

Bạn dễ dàng nhận thấy rằng `adopters` được khai báo là `public`. Các biến được khai báo là `public` thì sẽ tự động có hàm `getters`.
Tuy nhiên, đối với một `array` thì cần phải truyền thêm key để trả về một giá trị đơn.

> Để trả về được toàn bộ `array` qua hàm getter thì chúng ta cần tuỳ biến và chúng ta sẽ quay lại chủ đề này trong phần phát triển front-end cho ứng dụng.

## Function đầu tiên: mua một con thú cưng

Chúng ta định nghĩa một function để cho phép người dùng cuối có thể đặt mua con thú cưng mà họ thích. 

```
function adopt(uint petId) public returns (uint) {
  require(petId >= 0 && petId <= 15);

  adopters[petId] = msg.sender;

  return petId;
}
```

`Solidity` yêu cầu phải định nghĩa đầy đủ kiểu dữ liệu của input param và của return value; như function ở trên thì đều được khai báo là kiểu `uint`.

Đầu tiên, chúng ta sử dụng method `require()` để đảm bảo rằng `petId` nằm trong khoảng cho phép từ 0 đến 15 của array.

Sau đó, chúng ta gán địa chỉ cho `petId` tương ứng. Để lấy địa chỉ của người gửi hoặc địa chỉ của smart contract chúng ta sử dụng `msg.sender`.

## Function thứ hai: Lấy toàn bộ giá trị của mảng `adopters`

Ở trên chúng ta đã đề cập rằng mặc định thì các hàm `getters` chỉ trả về giá trị đơn lẻ. Giao diện của ứng dụng yêu cầu cần phải
cập nhật status của toàn bộ 16 chú thú cưng hiện có nên nếu gọi 16 lần thì khá bất tiện nên chúng ta định nghĩa hàm getter sau để lấy toàn bộ 16 adopters.

```
function getAdopters() public returns (address[16]) {
  return ;
}
```

Vì `adopters` đã được khai báo nên chúng ta chỉ cần đơn giản là trả về `adopters` array. Điểm chú ý ở đây là cần khai báo kiểu trả về của hàm getter là `address[16]` 
để có thể lưu trữ toàn bộ 16 địa chỉ của các chú thú cưng "yêu quí" của chúng ta. Nếu có chú nào bị lạc mất thì buồn lắm lắm! :((

## Biên dịch và migration smart contract

### Biên dịch

*Solodity* là một ngôn ngữ biên dịch, nên chúng ta cần dịch mã nguồn ra bytecode để có thể chạy trên `EVM`. Nôm na là chúng ta dịch các đoạn mã Solidity ra mã bytecode để EVM có thể hiểu và chạy được.
Rồi deploy lên blockchain để chúng ta có thể tương tác với smart contract đã định nghĩa.

Trước tiên, chúng ta mở một cửa sổ console mới rồi chạy lệnh `testrpc` để khởi động một blockchain ở local. Màn hình output trên console có thể sẽ như sau

```
EthereumJS TestRPC v4.1.3 (ganache-core: 1.1.3)

Available Accounts
==================
(0) 0xe32e6002945ce7ea48e7bd923ff5b06c1e7911fc
(1) 0x6500399e3bfb1b9de26f3c7c5d16c5d1464b9090
(2) 0x198278955ce07b307cfb95d70f0701dc59678e91
(3) 0xf6829f0ea570f531719bfb41c84251e10dabe1e2
(4) 0x99d45822dcac8ba6fea0c763eba7572d5141d95b
(5) 0xf15c92af140da40089fd363f1878ec0e53fa3824
(6) 0x47d907c367f13a81962e79ddd885d5d35e9eda45
(7) 0x9aa2fb2637b4dcf0ef304f5ca84d9472ef29fef6
(8) 0x67a828afaf1ac817926e8a2da7c3ed6f572509ef
(9) 0xaa5087b834825d42de88af8c66b7015da79f93e7

Private Keys
==================
(0) 094decef037de42b2dea807d32640863d9c2ab9cff520da687836e609be72d47
(1) 6031e682e974a1fdc9fb80edcbcb9181d607ebb002158a66fba7cbd00de82bef
(2) feb5e58e50a18fa2b1ff0a898358274e0db6830c739f08b1985ec8588af352b3
(3) 3ddbef7547c71d28dc847e6b85217ef6de8935dbf3d7054f75c55aedd24a39bb
(4) 3fa3ee86e67a0f37bbf93be15cce22f6f9d923a69eeb3611383450d032f30929
(5) 72527b5043da3722dba4067d61c11c6dc374f5e07a80bdcb00f530b261b71ef2
(6) 378880889ac1df868c316f1b337628675f674fa911b6dcd6924fff913c94936e
(7) c4c1734fe176e5e19156c004e748f009381c0a80e2cb82ddf2d332c8477c6c87
(8) 8299aaf8e4cbe43dcb2a3f76f5f25e736d0afe980847f161a70bbef6723eba3b
(9) 0cf8e2f65f90c9a7164c1d048911669d7f1eed86e134e60b1028a5c5683de747

HD Wallet
==================
Mnemonic:      actress gadget attack tribe brave solve effort youth boy enrich train island
Base HD Path:  m/44'/60'/0'/0/{account_index}

Listening on localhost:8545
```

Sau đó quay lại cửa số console của *pet-shop* rồi chạy lệnh `truffle compile` để biên dịch smart contract ra bytecode trong thư mục `build`.
Nếu kết quả output ra console như sau thì mọi việc đang tốt đẹp, nếu không thì bạn kiểm tra lại xem có lỗi gì không?

```
Compiling ./contracts/Adoption.sol...
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts
```

### Migration

Sau khi đã biên dịch thành công smart contract ở bước trên, bây giờ chúng ta thực hiện migration smart contract lên blockchain.
Migration là một mã code để deploy và thay đổi trạng thái của contract ứng dụng của chúng ra. Ở lần migration đầu tiên thì chỉ đơn giản là deploy code mới lên blockchain.
Còn các lần migration sau, ngoài việc deploy contract mới thì còn bao gồm cả việc migration các dữ liệu cũ cho phù hợp với smart contract mới.

Mặc định, chúng ta để ý thấy đã có một file `1_initial_migration.js` với nội dung sau

```
var Migrations = artifacts.require("./Migrations.sol");

module.exports = function(deployer) {
  deployer.deploy(Migrations);
};
```

File này dùng để deploy smart contract `Migrations.sol` đặc biệt đã được định nghĩa sẵn để quản lý việc migration cho các smart contract mà chúng ta tuỳ biến theo nhu cầu của project.
Migration được thực hiện tuần tự theo các bước sau đây cơ bản chung sau đây:

* Import smart contract artifact từ thư mục `build`
* Export một anonymous function với một tham số là `deployer`
* Yêu cầu deployment smart contract đã import ở trên bằng cách gọi hàm `deployer.deploy(<<CONTRACT_NAME>>)`

Để deploy contract tuỳ biến của ứng dụng thì thông thường chúng ta sẽ tạo một script file khác. Ở đây, chúng ta sẽ tạo ra file `2_deploy_contracts.js` như sau

```
var Adoption = artifacts.require("./Adoption.sol");

module.exports = function(deployer) {
  deployer.deploy(Adoption);
};
```

Sau đó chạy lệnh `truffle migration` để deploy smart contract lên local blockchain, output trên console như sau thì chắc là mọi thứ đang ổn

```
Using network 'development'.

Running migration: 1_initial_migration.js
  Deploying Migrations...
  ... 0x9d85e3633c795661e33d0a42cfc568bfc5a2c9e5f24de52c5a3596957447a284
  Migrations: 0x156eaa7e181125379c0e37653300c83fda934c0c
Saving successful migration to network...
  ... 0xec80ef7ddb31bd2dbb50ece7abec01e8d12804012f9fbf03a42b84882e493fff
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Adoption...
  ... 
  Adoption: 0x740c353b61dc5285065164ba3e6881485efa644e
Saving successful migration to network...
  ... 0xbb539b696da8d4986d85a937a9dd34f81b3a6632bcda462a111816df5a676068
Saving artifacts...
```

Wow, phải ăn mừng thôi; chúng ta vừa deploy thành công smart contract trên local blockchain testnet.
Bạn có thể để ý thấy địa chỉ của `adoption` smart contract là `0x740c353b61dc5285065164ba3e6881485efa644e` 
còn giá trị `0xec80ef7ddb31bd2dbb50ece7abec01e8d12804012f9fbf03a42b84882e493fff` chính là transaction id của giao dịch. 
Về sau chúng ta có thể check các thông tin tương tự trên các public network thông qua [etherscan.io](https://etherscan.io)

Bước tiếp theo chúng ta sẽ thao tác với smart contract để đảm bảo là nó sẽ hoạt động đúng như mong đợi.

## Kiểm thử Smart Contract

Truffle đã đi kèm với những công cụ cho phép chúng ta thực hiện test smart contract. Chúng ta có thể viết code kiểm thử smart contract bằng JavaScript hoặc bằng Solidity. 
Hôm nay, chúng ta sẽ sử dụng Solidity để viết code test.

Đầu tiên, chúng ta tạo file `TestAdoption.sol` trong thư mục *test* như sau.

```
pragma solidity ^0.4.11;

import "truffle/Assert.sol";
import "truffle/DeployedAddresses.sol";
import "../contracts/Adoption.sol";

contract TestAdoption {
  Adoption adoption = Adoption(DeployedAddresses.Adoption());

}
```

Bắt đầu với 3 dòng lệnh import các thư viện cần thiết cho việc kiểm thử.

* `Assert.sol`: để kiểm tra kết quả có đúng mong đợi hay không rồi đưa ra kết quả test là pass hay fail.
* `DeployedAddresses.sol`: Khi chạy code test, Truffle sẽ deploy một một smart contract mới trên *TestRPC* 
cho mục đích test và thư viện này cũng là một smart contract cho phép chúng ta lấy được địa chỉ của smart contract đã đuợc deploy.
* `Adoption.sol`: chính là smart contract chúng ta muốn kiểm thử.

Để demo code test thì chúng ta viết 3 method test đơn giản như sau.

```
  function testUserCanAdoptPet() {
    uint returnedId = adoption.adopt(8);

    uint expected = 8;

    Assert.equal(returnedId, expected, "Adoption of pet ID 8 should be recorded.");
  }

  function testGetAdopterAddressByPetId() {
    address expected = this;

    address adopter = adoption.adopters(8);

    Assert.equal(adopter, expected, "Owner of pet ID 8 should be recorded.");
  }

  function testGetAdopterAddressByPetIdInArray() {
    address expected = this;

    address[16] memory adopters = adoption.getAdopters();

    Assert.equal(adopters[8], expected, "Owner of pet ID 8 should be recorded.");
  }
```

Sau đó, chạy lệnh `truffle test` để execute code test và kết quả test sẽ được output ra console như sau nếu như mọi thứ hoạt động đúng như mong đợi.

```
Compiling ./contracts/Adoption.sol...
Compiling ./test/TestAdoption.sol...
Compiling truffle/Assert.sol...
Compiling truffle/DeployedAddresses.sol...

  TestAdoption
    ✓ testUserCanAdoptPet (108ms)
    ✓ testGetAdopterAddressByPetId (100ms)
    ✓ testGetAdopterAddressByPetIdInArray (164ms)


  3 passing (873ms)v
```

Ở phần 2, tôi sẽ ghi lại các bước phát triển front-end tương tác với smart contract đã deploy ở trên.
 