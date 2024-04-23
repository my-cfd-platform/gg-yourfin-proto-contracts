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







# Services description

## AccountsIntegrationGrpcService

This service has methods:
* CreateClientAccount - creates an account for a user;
