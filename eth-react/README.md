# Ethereum with React -- HelloWorld

## Five Steps
    1. Start Ganache (listening on http://127.0.0.1:7545)
    2. Unlock MetaMask with Local Network (create a customized private RPC network that listen on port 7545)
    3. Switch Accounts (add a private key from Ganache to MetaMask account)
    4. Every time we change our contract code, we have to execute `truffle compile`, and then `truffle migrate`
    5. npm run start

In `contracts` folder, we create our contract `HelloWorld.sol`

In `App.js`:
```JS
import helloWorldContract from '../build/contracts/helloWorld.json'
```

In `HelloWorld.sol`:
```Solidity
// Version Pragma. This allows us to specify which version of Solidity
// we want to compile with so as to not introduce breaking changes in future versions.
pragma solidity ^0.4.18;

contract HelloWorld {
  // Define the hello as type string
  string hello;

  // This is our constructor and runs when the contract is executed
  function HelloWorld() public {
    hello = "Hello World!";
  }

  // We can change our hello message with this.
  // We underscore function arguments to differentiate between arguments and global variables.
  // This is considered debatable, but we follow popular convention here.
  function setHello(string _hello) public {
    hello = _hello;
  }

  // Main function
  function getHello() public view returns (string) {
    return hello;
  }
}

```


## Work with `App.js`

What we need ...
1. Import everything we need
```JS
import React, { Component } from 'react'
import HelloWorldContract from '../build/contracts/HelloWorld.json'
import getWeb3 from './utils/getWeb3'
// Import our components.
import ContractInput from './components/ContractInput'
import Modal from './components/Modal'

// Import our fonts and CSS.
import './css/roboto.css'
import './css/rubik.css'
import './css/milligram.min.css'
import './App.css'
```
2. Set our initial state
```JS
    // Change initial states and add new ones here.
    this.state = {
      hello: "I'm waiting to say hello...",
      contractAddress: "Waiting on contract address...",
      modal: 0,
      instance: null,
      web3: null
    }
```
3. Get web3
```JS
componentWillMount() {
    // Get network provider and web3 instance.
    // See utils/getWeb3 for more info.
    getWeb3
    .then(results => {
      this.setState({
        web3: results.web3
      })
      // You can see the what is in web3
      console.log(results.web3)

      // Instantiate contract once web3 provided.
      this.instantiateContract()
    })
    .catch(() => {
      console.log('Error finding web3.')
    })
  }
```

4. Instantiate our contract
```JS
instantiateContract() {
    const contract = require('truffle-contract')
    const helloWorld = contract(HelloWorldContract)
    let helloWorldInstance
    helloWorld.setProvider(this.state.web3.currentProvider)

    helloWorld.deployed().then((instance) => {
      helloWorldInstance = instance
      this.setState({ instance: helloWorldInstance })

      console.log(helloWorldInstance)
      /* "TruffleContract" object with
              abi: contract abi
              address: contract address
              getHello(): contract function
              setHello(): contract function
              send()
              sendTransaction()
              transactionHash
              ...
      */

      // Display the address of our smart contract.
      this.setState({ contractAddress: helloWorldInstance.address })

      /* Display our default Hello World message from HelloWorld.sol
       * Once you change the default message, you will need to truffle migrate --reset
       * to see the original message again.
       */
      return helloWorldInstance.getHello()  // call the contract function in JS,
                                            // then return the result to update our state

    }).then((result) => {
      return this.setState({ hello: result })
    })
  }
```
5. Render our HTML/compononets
```JS
render() {
    return (
      <div className="App">
        <div className="top-bar">
          <a href="#" className="title-link">🍕 Ethereum and React</a>
          <div className="notice-box">Looks like Truffle React Box is up and running 👍👍</div>
        </div>

        <main className="container">
          <h1>Hello World</h1>
          <h2>Our First Smart Contract</h2>
          <div className="contract-status">
            <p>If your contracts compiled and migrated successfully, we willll show your contract address and the hello message below.</p>
            <div>Your contract address is: <span className="contract-address">{this.state.contractAddress}</span></div>
          </div>
          <p className="message">The hello message from your contract is: <strong className="hello-world">{this.state.hello}</strong></p>
          <ContractInput state={this.state} updateHello={this.updateHello} initModal={this.initModal} />
        </main>
        <Modal modal={this.state.modal} />
      </div>
    );
  }
```

### In ContractInput.js
```JS
  // render
  // Get the user input, and set it to this.message
  render() {
    return (
      <form className="hello-form" onSubmit={(e) => this.submit(e)}>
        <input ref={(input) => this.message = input} type="text" className="hello-input" placeholder="Hello World text to change" />
        <button type="submit" value="Submit" className="button-submit js-button-submit">Submit</button>
      </form>
    )
  }

  // what is submit do ?
  submit(e) {
    e.preventDefault();
    const message = this.message.value;
    const state = this.props.state;
    const instance = state.instance;

    const setHelloRequest = async () => {
      const result = instance.setHello(message, { from:  state.web3.eth.accounts[0]}).then((result) => {
        this.props.initModal(0);
        return instance.getHello()
      });
      return result
    }

    const getHelloRequest = async () => {
      this.props.initModal(1);
      const result = await setHelloRequest();
      this.props.updateHello(result);
    }

    getHelloRequest();
  }
```
