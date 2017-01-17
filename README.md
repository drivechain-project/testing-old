Testing Drivechains
===================

Setup
-----
- Create directories to store the mainchain and sidechain repositories
```
mkdir -p ~/drivechain/{mainchain,sidechain}
```

- Create directories for mainchain and sidechain datadir
```
mkdir -p ~/drivechain/data/{maindata,sidedata}
```

`Note: this guide will assume that you have created the same directories`

### Mainchain (bitcoin) setup
- Clone repository and checkout mainchain branch
```
cd ~/drivechain/mainchain/
git clone https://github.com/drivechain-project/bitcoin.git
git fetch
git checkout mainchain
```

- Build
```
./autogen.sh
./configure
make
```

- Configuration file

Copy the main.conf configuration file from the [Drivechain testing repository](https://github.com/drivechain-project/testing)

to ~/drivechain/data/maindata/bitcoin.conf

### Sidechain setup
- Clone repository and checkout sidechain branch
```
cd ~/drivechain/sidechain/
git clone https://github.com/drivechain-project/bitcoin.git
git fetch
git checkout sidechain
```

- Build
```
./autogen.sh
./configure
make
```

- Configuration file

Copy the side.conf configuration file from the [Drivechain testing repository](https://github.com/drivechain-project/testing)

to ~/drivechain/sidedata/bitcoin.conf


Running unit tests
------------------
From the mainchain directory (~/drivechain/mainchain/)
run all of the unit tests:
```
make check
```

or specify the sidechaindb unit test:
```
src/test/test_bitcoin --log_level=all --run_test=sidechaindb_tests
```

Manual testing
--------------
### Get everything started
- Start mainchain (bitcoin)

From the mainchain directory (~/drivechain/mainchain/)
`replace $USERNAME with your username!`
```
./src/qt/bitcoin-qt --datadir=/home/$USERNAME/drivechain/maindata/
```

- Start sidechain

From the sidechain directory (~/drivechain/sidechain/)
`replace $USERNAME with your username!`
```
./src/qt/bitcoin-qt --datadir=/home/$USERNAME/drivechain/sidedata/
```

Both clients (sidechain & mainchain) should start in regtest mode, note the
blue icon. If not, you need to make sure you saved the configuration files
correctly and started the clients with the proper datadir specified.


- Mine 101 blocks on the mainchain to get coins

Open the debug window, switch to the console tab and enter:
```
generate 101
```

### Create deposit from mainchain to sidechain

- Deposit to sidechain

Go to the send page of the `mainchain` bitcoin-qt client, click send (sidechain)
which is located at the bottom of the page in the same row of buttons as the
normal Send button. The sidechain deposit dialog will pop up.

Get a deposit address from the sidechain tab of the `sidechain` bitcoin-qt.

![](SidechainPageDeposit.png)

Paste the sidechain deposit address into the deposit dialog open on the
`mainchain` client.

Enter the amount you wish to deposit. In this example we will send 7 BTC to the
sidechain.

![](SidechainDepositDialog.png)

Click the deposit button. Upon success the results will be displayed, or an
error message if there were issues.

![](SidechainDepositResultMessage.png)

Generate a block on `mainchain` to add the deposit to the DB.
```
generate 1
```

- Generate enough blocks on sidechain to clear the deposit

From the console tab of the `sidechain` debug window:
```
generate 101
```
You should see that 7 BTC have been deposited from the bitcoin mainchain into
the sidechain address that you specified.

### Withdraw from sidechain to mainchain

- Withdraw from sidechain

With the `sidechain` client open, visit the sidechain tab.

Click withdraw, enter the `mainchain` bitcoin address that you would like to
withdraw to and the amount to withdraw. In this example we will withdraw 3 BTC
from the sidechain and send it back home to bitcoin.

![](SidechainPageWithdraw.png)

Click the withdraw button to broadcast it.

### Trigger WT^ creation, verification and payout

- Generate blocks on sidechain to create and broadcast WT^

`WT^ = grouped withdrawal transactions to be validated by mainchain`

- Generate enough blocks on the `sidechain` client to trigger the creation of
WT^, which will be broadcast automatically to the bitcoin mainchain. If you are
following along and made a deposit to SIDECHAIN_TEST then the WT^ creation will
be triggered every 300 blocks.

Example (if current block height = 150)
```
generate 150
```

Generate a block on the mainchchain to add the WT^ to the DB and start the
verification process
```
generate 1
```

- Finally, generate blocks on mainchain to trigger WT^ payout

Generate enough blocks on the mainchain to increment the workscore of WT^ up to
the minimum workscore for the sidechain. If everything worked out correctly you
will see your `sidechain` withdraw deposited into the `mainchain` bitcoin
address that you specified.

Example (SIDECHAIN_TEST minWorkScore = 100)
```
generate 100
```
![](SidechainWithdrawReceived.png)

If you received the 3 (~2.9 with fee removed) BTC withdraw from the mainchain,
it worked.
