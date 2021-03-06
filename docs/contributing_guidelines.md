# Contributing Guidelines
## Steps to contribute
- Create branch for each feature
- Follow TDD for each component/store
- Use flow to define types
- DO NOT use `npm` to install packages, not even `npm 5`. Always use `yarn` to install dependencies
- Tests are run by default when you try to push. If tests fail then code will not be pushed, not even to local branch.
- Push will fail even if there are any lint errors
- Raise pull request to merge any change to master
- Fill Pull request template properly while raising the PR

## Process
- PR template is made for submitting PR
- PR is `squashed and merged`
- Tests are run before each push
- Files are automatically formatted by prettier before committing
- Hooks for pre-commit and pre-push are in `package.json`


### How to test screens without any external dependency

#### Create connection without generating build and clicking deep link

- Go to config store and find line with something like this `...baseUrls[SERVER_ENVIRONMENT.STAGING]`. Change `STAGING` with `DEVELOPMENT` or `SANDBOX`.
- In deep-link/index.js. Find method `onDeepLinkData` and add this line at the start of method
```js
this.props.deepLinkData('6a3e8fd0')
```
The string that we have passed inside `deepLinkData` comes from email. Contact other developers for email account access.

#### Proof request screen

Add these lines to import statement of Home screen
```js
// TODO Remove these lines after testing is done
import { proofRequest, proofRequestId } from '../../__mocks__/static-data'
import { proofRequestRoute } from '../common/route-constants'
import { proofRequestReceived } from '../proof-request/proof-request-store'
```

Add this method to Home screen component
```js

  // TODO Remove this whole function, this is for testing purpose
  componentDidMount() {
    // $FlowFixMe
    this.props.proofRequestReceived(
      proofRequest.payload,
      proofRequest.payloadInfo
    )
    setTimeout(() => {
      this.props.navigation.navigate(proofRequestRoute, { uid: proofRequestId })
    }, 1000)
  }
```

Bind methods to Home props
```js

// TODO: Remove below two line, Only for testing purpose
const mapDispatchToProps = dispatch =>
  bindActionCreators(
    {
      proofRequestReceived,
    },
    dispatch
  )

export default connect(mapStateToProps, mapDispatchToProps)(DashboardScreen)

// TODO: Un-comment this line after testing is done
//export default connect(mapStateToProps)(DashboardScreen)
```

Go to `mapStateToProps` in proof-request.js and find logoUrl
```js
// TODO: Un-comment this line and remove line next to this line
// logoUrl: getConnectionLogoUrl(state, remotePairwiseDID),
logoUrl: 'https://image.flaticon.com/icons/png/128/174/174851.png',
```

In proof-store.js. Import static-data for proof
```js
// TODO Remove below import, only for testing
import { proofWith10Attributes,homeAddressPreparedProofWithMissingAttribute } from '../../__mocks__/static-data'
```

Find the call to `prepareProof` in `generateProofSaga`. Comment out the actual call to native bridge and use this instead
```js
const preparedProofJson=JSON.stringify(homeAddressPreparedProofWithMissingAttribute)
```

Find the call to `generateProof` in `generateProofSaga`. Comment out the actual call to native bridge and use this instead
```js
// TODO Remove below line and un-comment above line, only for testing
const proofJson = JSON.stringify(proofWith10Attributes)
```

#### Paid Claim Offer Accepted screen
Add these lines to import statement of Home screen
```js
// TODO Remove these lines after testing is done
import {claimOfferId,paidClaimOffer} from '../../__mocks__/static-data'
import { claimOfferRoute } from '../common/route-constants'
import {claimOfferReceived} from '../claim-offer/claim-offer-store'
```

Add this method to Home screen component
```js

  // TODO Remove this whole function, this is for testing purpose
   componentDidMount() {
    // $FlowFixMe
   this.props.claimOfferReceived(paidClaimOffer.payload,paidClaimOffer.payloadInfo)
   setTimeout(() => {
     this.props.navigation.navigate(claimOfferRoute, { uid: claimOfferId})
   }, 4000)
  }
```

Follow the process mentioned below in Claim offer screen.

#### Claim Offer Accepted screen

Add these lines to import statement of Home screen
```js
// TODO Remove these lines after testing is done
import {claimOfferId,claimOffer} from '../../__mocks__/static-data'
import { claimOfferRoute } from '../common/route-constants'
import {claimOfferReceived} from '../claim-offer/claim-offer-store'
```

Add this method to Home screen component
```js

  // TODO Remove this whole function, this is for testing purpose
   componentDidMount() {
    // $FlowFixMe
   this.props.claimOfferReceived(claimOffer.payload,claimOffer.payloadInfo)
   setTimeout(() => {
     this.props.navigation.navigate(claimOfferRoute, { uid: claimOfferId})
   }, 4000)
  }
```

Bind methods to Home props
```js
//TODO remove this, only for testing purpose
const mapDispatchToProps = dispatch =>
  bindActionCreators(
    {
      claimOfferReceived,
    },
    dispatch
  )

export default connect(mapStateToProps, mapDispatchToProps)(DashboardScreen)
//export default connect(mapStateToProps)(DashboardScreen)
```

Go to `mapStateToProps` in claim-offer.js and find logoUrl
```js
 // TODO: Un-comment this line and remove line next to this line
  //const logoUrl = getConnectionLogoUrl(state, claimOfferData.remotePairwiseDID)
  const logoUrl ='https://image.flaticon.com/icons/png/128/174/174851.png'
```

In claim-offer-store.js 

Go to claimOfferAcceptedSaga use hardcoded userPairwiseDid and parsedClaimRequest.
Also Comment out the sendMessage native bridge call
```js


// TODO use hardcoded userPairwiseDid, only for testing
// const userPairwiseDid: string | null = yield select(
  //   getUserPairwiseDid,
  //   remoteDid
  // )
  const userPairwiseDid: string | null = 'userPairwiseDid'

   //TODO uncomment these lines and remove hardcoded parsedClaimRequest
      // const stringifiedClaimRequest: string = yield call(
      //   generateClaimRequest,
      //   remoteDid,
      //   indyClaimOffer,
      //   poolConfig
      // )
      // TODO:KS Add error handling if claim request parse fails
      // const parsedClaimRequest: IndyClaimRequest = JSON.parse(
      //   stringifiedClaimRequest
      // )
      
      const parsedClaimRequest: IndyClaimRequest = {
        blinded_ms: {
          prover_did: 'prover_did',
          u: 'u',
        },
        issuer_did: 'issuer_did',
        schema_seq_no: 36,
      }

```
#### To trigger claim offer success pop up

  in claim-request-modal.js add the following code 


```js
    componentDidMount() {
    // if modal is opened when it was unmounted or freshly mounted
    // this.toggleModal(this.props.claimRequestStatus)

    setTimeout(() => {
      this.setState({isVisible: true})
    }, 4000)
  }

```

#### Test credentials and proofs on simulator

In componentDidMount of push-notification.js. Add below code at end

```js
this.props.fetchAdditionalData({
      // forDID can be extracted from redux logs inside connection reducer, it is named as `identifier`
      forDID: 'Mua2EG3rZphtwj88LWRpcw',
      // uid can get from network request log in verity-ui and checking the response for that particular credential or proof request
      uid: 'njaxymv',
      // types supported are credOffer cred, proofReq
      type: 'credOffer',
      // can be extracted from same place as forDID, key named as senderDID
      remotePairwiseDID: '4yKr2YScm8o7nRXRvbHvzA',
      // can be kept the same
      senderLogoUrl: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQEc_lUYZc74j-ArO9tldKiyLusNRGQE1X4dBR8Yz-J1ZcrAduLYg',
    })
```

### How to enable Android RUST logs
- un-comment this line in android/app/build.gradle
```js
// compile group: 'com.mobidevelop.robovm', name: 'robovm-rt', version: '2.3.4'
```
- un-comment these line in `MainActivity.java`
```java
//import libcore.io.ErrnoException;
//import libcore.io.Libcore;
...
//        try {
//            Libcore.os.setenv("RUST_LOG", "TRACE", true);
//        } catch (ErrnoException e) {
//            e.printStackTrace();
//        }
```

### How to enable RUST logs in ios
- Open connectme workspace in xcode
- From Menu Product -> Scheme -> Scroll down to bottom -> Edit Scheme
- A window will open. On left side of this window, select "Run", on right side click on "Arguments"
- Expand "Environment Variables"
- Check checkboxes for RUST_LOG and RUST_BACKTRACE

## How to know what tests to write

Check this small video https://drive.google.com/open?id=1t83ZTe4RdgozuIRs6Pi140fU7cIp-8wx
