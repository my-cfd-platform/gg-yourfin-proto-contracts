# Integration to platform documentation.

## Overview

Integration to platform contains the following components:
* AccountsIntegrationGrpcService - gives ability to make operations with accounts;
* AuthenticationIntegrationGrpcService - gives ability to authenticate users without extra hassle for a user by asking users to enter username and password;
* TradingInfoIntegrationGrpc - gives ability to receive information about trading objects such as:
  - Active positions;
  - Pending orders;
  - Trading history;
* TradingSettingsIntegrationGrpcService - gives ability to read information about trading settings of platform;
* TransactionHistoryIntegrationGrpcService - gives ability to read information about operations which affect balance;
* CallbacksIntegrationGrpc - gives ability to receive information about events which happen in the platform in realtime;

## Important things to consider:
* TraderId - is an identification of a user in the platform. Every user can have several trading accounts; Every object (except objects of TradingSettingsIntegrationGrpcService) belongs to some user (trader) hence it has a TraderId.
* AccountID - is an identification of a trading account. Single trader with TraderId can have several accounts;
* ApiKey - is a key which gives access to the trading platform for a 3rd party company which integrates to the platform.

## ProcessId field

Field ProcessId is a part of create domain object request. It gives ability to retry request as many time, as requires to get the response.
For example. We want to create new trading account. Request requires other domain objects fileds + ProcessId

```json
{
  "TraderId": "XXXXXX-YYYY-ZZZ",
  "Currency": "USD",
  "TradingGroupId": "Main",
  "ProcessId": "2021-04-01T12:00:13.123456"
}
```

We can send this payload as many times as required until we get the Positive response that account is created. 
It's important to use the same ProcessId if request is failed and application is retrying the request. 

To generate unique processId - it's enough to generate a timestamp string with microseconds precision.


# Trading platform settings

Trading platform settings can be read by TradingSettingsIntegrationGrpcService.

The structure of the objects of trading platform is following:

## Trading Instrument;

Described as a protobuf structure TradingInstrumentGrpcModel;
* Id - id of instrument;
* Name - human friendly name of the instrument;
* Digits - precision of the instrument. Example: 3 means 1.123; 5 means. 1.12345;
* Base - Base asset of instrument;
* Quote - Quote asset of instrument;
* TradingDisabled - means that instrument is not used for trading for now;
* TradingInstrumentDayOffs - an array of time intervals when instrument is not trading;

### TradingInstrumentDayOffs structure
* DateFrom - date of the week from. 0=Sunday;
* TimeFrom - time in format hh:mm::ss;
* DateTo - date of the week from. 0=Sunday;
* TimeTo - time in format hh:mm::ss;

### Trading Groups

Trading group - is an entity which contains several profiles:
* TradingProfile - Profile which contains settings related to trading operations;
* SwapProfile - Profile which contains settings related to swap operations;

As well trading group has fields:
* Name - name of the group;
* TradingDisables - if true - accounts which are related to this trading group can not open new positions.

Every trading account is attached to a trading group

### Trading profile fields (TradingProfileGrpcModel)

* Id - id of profile;
* MarginCallPercent - Value from 0..1; todo!("Double check it")
* StopOutPercent - Value from 0..1; todo!("Double check it")
* array of trading instruments with their settings, related to trading profile;


### Instruments, related to trading profile settings (TradingProfileInstrumentGrpcModel)
* Id - id of instrument;
* MinOperationVolume -  minimal volume during the open position operation in Collateral currency of account; todo!("Double check it")
* MaxOperationVolume - maximum volume during the open position operation in Collateral currency of account; todo!("Double check it")
* OpenPositionMinDelayMs - minimal delay in milliseconds which is applied during the open position operation;
* OpenPositionMaxDelayMs - maximal delay in milliseconds which is applied during the open position operation;
* Leverages - array of leverages, which can be selected to open position. 100 means 1:100.
* StopOutPercent - if not null - overrides StopOut value from TradingProfile;

### Swap profile fields
* Id - id of profile;
* Instruments - list of instruments which swap settings;

### Swap instrument settings (SwapProfileInstrumentGrpcModel):
* InstrumentId - id of instrument;
* Long - todo!("Check what is there")
* Short - todo!("Check what is there")

### Swap Schedule (SwapScheduleGrpcItem)
* SwapProfileId - id of swap profile;
* Items - array of schedule items of swap profile;

### Schedule items of swap profile (SwapScheduleGrpcItem)
* DayOfWeek - 0-Sunday;
* Time  - moment of time when swap must be applied. Format: hh:mm:ss
* Amount - amount of swaps to be applied from Swap profile;

### GetServerInfo
* ServerTime - returns server time and Timezone in RFC 3339 format;

# AccountsIntegrationGrpcService

## CreateClientAccount
Creates client account.
### Input parameters
* TraderId - ID of trader;
* Currency - account collateral currency;
* ProcessId - unique ID of process. Please read ProcessId field usage.
* TradingGroupId - id of trading group which account is going to be created;
* Password - optional field. If provided - client can authenticate to the platform using the same 

If account is created the response would have Ok Status and Account Model (AccountsIntegrationClientAccountGrpcModel)

## AccountsIntegrationClientAccountGrpcModel fields
* ApiKey - authentication api key
* TraderId - id of trader was used at accounts registration request;
* AccountId - id of account generated by trading platform;
* Currency - currency of account;
* Balance - account balance;
* TradingGroupId - id of Trading group;
* CreatedAt - timestamp in microseconds;
* UpdatedIt - time of last update of any fields of the account entity;
* TradingDisabled - flag - if trading is disabled for the account;

## UpdateClientAccountBalance
Can be used to increase or decrease account balance;
### Input parameters
* ApiKey - authentication api key
* TraderId - id of trader was used at accounts registration request;
* AccountId - id of account generated by trading platform;
* Delta. >0 - balance would be increased. <0 - balance would be decreased;
* Comment - text comment of the operation;
* ProcessId - unique Id of balance update process;
* AllowNegativeBalance - is used at the case when we decrease balance. If true - does not allow account balance be negative after operation;
* ReferenceTransactionId - Text field. Useful - if balance change operation has a linked TrancationId of external 3rd party system. For instance - if we increase balance as a result of a payment system deposit operation - most probably the deposit operation has TrancationId of related payment system. This field can be used to keep that reference.

### Result
As a result - if not failed - returns
* OperationId - id of operation inside trading platform;
* AccountModel - new state of account (AccountsIntegrationClientAccountGrpcModel);

## GetClientAccounts
returns list of accounts, related to a trader;
### Input parameters
* ApiKey - authentication api key;
* TraderId - id of trader was used at accounts registration request;
### Result
List of accounts

## UpdateAccountTradingDisabled
Gives ability to enable/disable trading features at account level
### Input parameters
* ApiKey - authentication api key;
* TraderId - id of trader was used at accounts registration request;
* AccountId - id of account;
* TradingDisabled - true - disables the trading. false - enables the trading;

## AuthenticationIntegrationGrpcService
Gives the abilty to authenticate user to make sure the user would not type password when connecting to the platform.

## CreateOrUpdateTraderCredentials
Create client credentials, giving ability to authenticate user with the same UserName (email) and password.
If credentials are exists - username and password would be updated;
### Input parameters:
* ApiKey - authentication api key;
* TraderId - id of trader;
* Email - email which would be used at the moment of authentication;
* Password - password which would be used at the moment of authentication;

## GeneratePlatformRedirectUrl
Generates redirect url to the trading platform which allows to skip user authentication flow;

### Input parameters:
* ApiKey - authentication api key;
* TraderId - id of trader;
* AccountId - id of account, which would be selected once client is redirected;
### Result
* RedirectUrl - in case of success;


## CallbacksIntegrationGrpc
Allows to get updates in realtime. It's highly recommeded to do requests in an infinite loop without any delays between requests to make sure that data is delivered as soon as possible. Server would do all the delays if required.
As well, it's important to highligh that CallbacksIntegrationGrpc keeps queue to make sure that no messages are missed between requests.

### ConfirmationID
Every request requires to provide confirmationID of the previous response. 
Providing ConfirmationId makes sure that we do not miss any messages in case of connection problems.

## GetBidAsk
Returns list of BidAsk prices. There is a queue inside microservice to make sure, that no messages are missed between requests.
If requests are not performed during 60 seconds - queue is cleared.
### Input parameter
* ApiKey - authentication api key;
* ConfirmationId - confirmation id of previous response or empty - if this is a first request;
### Response
* ConfirmationId - must be populated to the next request;
* BidAsks - array of prices (BidAskGrpcModel)

## BidAskGrpcModel
* Instrument - id of instrument;
* DateTime - unix-microseconds;
* Bid - bid price;
* Ask - ask - price;

## GetChanges

Returns list of Orders changes There is a queue inside microservice to make sure, that no messages are missed between requests.
Queue is never cleared. If application which is reading queue is down for a long time - once application is up - no events are going to be missed.
### Input parameter
* ApiKey - authentication api key;
* ConfirmationId - confirmation id of previous response or empty - if this is a first request;
### Response
* ConfirmationId - must be populated to the next request;
* Changes - array of order changes if changes were enqueued between requests;

### ChangeGrpcModel

Shows what kind of change happend on platform

* ChaneType - shows if position was opened or closed;
* TraderId - id of trader;
* AccountId - id of account;
* Id - id of order/active position/History event;
