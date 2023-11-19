# Node-RED integration with Ampio
## tedee lock integration
### Intro
I use Node-RED integration to include basic operations on tedee locks in Ampio app. I have two tedee locks and this setup is used in the sample code, but it is straightforward to extend it to more.

The examples provided in [Ampio documentation](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/) had two problems for me:
1. They were using an ancient API (probably not bad in itself but being a tech person I always prefer to be up to date)
2. They were suited to using one device only and I had to put some work to make it work with two locks, so I decided to put a bit more work

### tedee component
tedee component is used to communicate with the cloud services of tedee controlling your locks. It periodically (every 20 seconds) reads the status (locked or unlocked or any intermediary statuses) and battery level of all locks under the account.

The component shows the command status (colorful dot on the component):
- green: operation on the lock (`open`, `close`) was successful
- blue: the lock status was read correctly
- red: some error occured (http error is provided in the description)

#### Environment variables
The component uses one env variable: `Personal Access Key`. Copy-paste there your PAK from [Tedee Portal](https://portal.tedee.com/personal-access-keys).

#### Inputs
The component has one input which is an object under `msg.payload` with two fields mapped:
- `command`, which can take value of `open` or `close` to operate on the lock; any other value will trigger lock status read
- `lockID`, which is a lock identifier (you can check the value in the tedee mobile app) and is used only for `open` and `close` commands

#### Outputs
The component has four outputs under `msg.payload` (each emitted independently):
1. An object of:
- `id`: id of the lock
- `state`: [numeric value](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/enums/lock-state.html) of the lock state
2. An object of:
- `id`: id of the lock
- `state`: [text value](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/enums/lock-state.html) of the lock state
3. An object of:
- `id`: id of the lock
- `batteryLevel`: numeric value (percentage) of the battery level

### Ampio preparation
In general, follow the instruction provided in [Ampio tutorial](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/). You will need to note down:
- tedee lock ID as indicated in the tedee app
- identifiers of the open/close flags created in SmartHomeManager
- the line flag identfier on the selected module to store the lock status
- mac address of the module on which `open` and `close` flags are configured
- flag identifiers on the selected module associated with `open` and `close` operations

### tedee-Ampio integration
There are two things you need to configure:
1. adjust number of Ampio IN components corresponding to the number of tedee locks you want to control (you can copy the ones included in the example and configure them accrodingly)
2. configure the `initialize` function component in `On Start` tab; for each lock you need to add three lines (or adjust existing ones in the sample):
```
flow.set("XXXX", {stateObjectID: "AAA", batteryLevelObjectID: "BBB", flagID: "N"});
flow.set("ampio/from/MAC/state/f/Y", {lockID: "XXXX", command: "open"});
flow.set("ampio/from/MAC/state/f/Z", { lockID: "XXXX", command: "close" });
```
where the meaning is as follows:
- `XXXX`: tedee lock ID as indicated in the tedee app (`9263` from [the example in the doc](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/))
- `AAA`, `BBB`: identifiers of the open/close flags created in SmartHomeManager (`1230` and `1231` from [the example in the doc](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/))
- `N`: the line flag identfier on the selected module (`1` from [the example in the doc](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/))
- `MAC`: mac address of the module on which `open` and `close` flags are configured (`819e` from [the example in the doc](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/))
- `Y`, `X`: flag identifiers on the selected module associated with `open` and `close` operations (`20` and `21` from [the example in the doc](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/))

### tedee API limitations
As per [documentation](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/howtos/authenticate.html) the authentication schema for non-interactive use case is to use Personal Access Key. You need to generate one at [Tedee Portal](https://portal.tedee.com/personal-access-keys). Please note that the key has a limited validity (you determine it while creating the key).

When authenticating with PAK there are no webhooks available and therefore it is required to periodically query the API for the lock status (which causes some delays in the refresh):
- [get all locks endpoint](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/endpoints/lock/get-all.html) has a limitation of 10 requests in 10 minutes
- [sync endpoint](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/endpoints/lock/sync.html) should not be called more than once every 10/20 seconds (two different values in the documentation)
- in particular it means that you cannot have two objects requesting lock/sync endpoint at the same time (if they are registered under the same PAK)

If you're receiving 429 http error it means that you're sending requests too fast or too often.

[The code samples](https://github.com/tedee-com/tedee-api-doc/tree/master/samples/cs) provided are outdated.

### Additional resources
- [Integration of the Ampio system with Node-RED](https://help.ampio.com/en/installer-tutorials/integration-of-the-ampio-system-with-node-red/)
- [Ampio: Integration with tedee lock](https://help.ampio.com/en/installer-tutorials/integration-with-tedee-lock/)
- [Tedee API documentation](https://tedee-tedee-api-doc.readthedocs-hosted.com/en/latest/)
- [Tedee API 1.32](https://api.tedee.com/swagger/index.html)
