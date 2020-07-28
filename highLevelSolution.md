Can write code in react es6 but writing in es5 will take time
In react, this is how I plan to to

Step # 1 Initialize client in construtor

devChallenge.const.js
export const URL = "ws://localhost:8011/stomp";

import {URL} from './devChallenge.const'

constructor() {
    super(props);
    this.client = Stomp.client(URL);
    this.state = {data: [], isError: false};
    this.getDataFromWebSocket = this.getDataFromWebSocket.bind(this);
    this.connectCallback = this.connectCallback.bind(this);
    this.compare = this.compare.bind(this);
    this.updateStateWithLatestData = this.updateStateWithLatestData.bind(this);
}

step # 2 Register eventListner on event 'ComponentDiMount' to get data from websocket

// sorting an array
function compare(a, b) {
  // Use toUpperCase() to ignore character casing
  const bidPriceA = a.lastChangeBid;
  const bidPriceB = b.lastChangeBid;

  let comparison = 0;
  if (bidPriceA > bidPriceB) {
    comparison = 1;
  } else if (bidPriceA < bidPriceB) {
    comparison = -1;
  }
  return comparison;
}

// update and merge coming cont from websocket
updateStateWithLatestData(item) {
    const itemValue = newData.body;

    let isAllKeyPresent = true;
    // check if required keys are present in the recieved object
    if (Object.keys(item).forEach((key) => {
        if (!['name','bestBid','bestAsk','lastChangeAsk','lastChangeBid'].includes(key)) {
            isAllKeyPresent =false;
            break;
        }
    }))
    if (!isAllKeyPresent) {
        return;
    }

    // add midPrice
    if (this.state.time >= 30) {
        itemValue['midPrice'] = (itemValue.bestBid + bestAsk)/2;
    }

    // replace old data with new data
    const isAlreadyPresent = this.state.data.filter((data) => data.name === item.name);
    if (!isAlreadyPresent) {
        // add new item to state
        const sortedData = this.state.data.push(item).sort(compare);
        this.setState({...this.state.data, data: sortedData})
    } else {
        const sortedData = this.state.data.concat(isAlreadyPresent).sort(compare);
        // update existing item with new value
        this.setState({...this.state.data, data: sortedData)
    }
}

function connectCallback() {
    this.subscribe = client.subscribe("/fx/prices", updateStateWithLatestData);
}

getDataFromWebSocket() {
    this.client.connect({}, connectCallback, function(error) {
        this.setState({...this.state, isError: true})
    })
}

componentDidMount() {
  this.interval = setInterval(() => this.setState({ time: Date.now() }), 1000);
  this.getDataFromWebSocket();
}

// render UI

render() {
    if (this.state.isError) {
        return (<>Error Occoured while connecting to Web socket</>)
    }
    return (
        <>
        // create columne headers
        this.state.data.map((item) => {
            return (
                <div key={item.name}>
                    <span>{item.name}</span>
                    <span>{item.bestBid}</span>
                    <span>{item.bestAsk}</span>
                    <span>{item.lastChangeAsk}</span>
                    <span>{item.lastChangeBid}</span>
                    {item.midPrice && new Sparkline(</span>).draw(item.midPrice)}
                </div>
            )
        })
        </>
    )
}

Step 3 Since we are are updating state whenever data is been recieved and changed, update is going to trigger using 
setState function resulting in another render



componentWillUnmount() {
  clearInterval(this.interval);
  // remove websocket event linstner too
  this.subscribe.unsubscribe();
}

