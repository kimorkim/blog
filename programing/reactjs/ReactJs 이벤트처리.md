#### React.Js 이벤트 처리
---
  이번에 친구와 같이 스도쿠 관련 스터디를 시작했다. 정확히는 오픈소스 스터디 그룹을 만들었고 그 시작이 스도구 생성 알고리즘 이긴하지만... 스도쿠 알고리즘 형태는 일단 구조를 잡아서 Example를 구현하기로 했는데 구현할때 이번에 React.Js를 활용해서 만들어 보기로 했다. 생각외로 더욱 신박하다. 기존에 jQuery스타일에 길들여진 나로썬 Es6 자바스크립트와 JSX가 상당히 이질적으로 느껴졌다. 하지만 인간은 역시 적응의 동물이랄가 계속 접하다보니 나름 익숙해졌다. 적당히 익숙해진 만큼 삽질한 내용들을 하나씩 공유 해보기로 마음먹었다... 매번 블로그를 작업하면서 꾸준히 글을 써야 겠다 하면서 작심삼일이 되곤 하는데 이번엔 최소한 한달에 한번씩은 썻으면 좋겠다. 이직 후 여유 시간이 늘었으니 가능할런지도 :)

##### 부모가 자식의 이벤트 받기
  처음 React.Js를 접하면서 가장 생소하게 느낌점이 이벤트 처리 이다. 분명 React.Js페이지에 가면 제공하는 이벤트 종류[^1]를 설명해주기는 하는데 단순히 제공되는 이벤트를 알려주는 수준에서 끝나고 있다. 부모 자식간의 이벤트 처리에 대해서는 자세한 설명이 없어서 애를좀 먹었다. 관련 라이브러리를 찾아 보았으나 EventEmitter처럼 처리 하는 방식의 라이브러리만 제공될뿐이었다. 어쩐지 AngularJs를 쓰는 기분이라 좀더 기본 React.Js적으로 처리할 방법을 생각해보고 해본결과 나름 깔끔한 방법이 나와서 공유 해본다.
  
  
  일단 다음과 아래와 같은 소스가 있다.
  
    import React, {Component} from 'react';
    import Box from './box';
    import './index.less';

    export default class Gameview extends Component{
        constructor() {
            super();
            let gameData = sdm.getGameData('random');
            this.state = {
                gameData: gameData
            };
        }    

        _makeGame() {
            let boxes = [];
            let boundClick = this.setValueEvent.bind(this);

            for(let i = 0; i < 9; i++) {
                boxes.push(
                    <Box
                        ref={i}
                        key={i}
                        position={i}
                        data={this.state.gameData.data[i]}
                        event={boundClick}
                    >
                    </Box>
                );
            }

            return boxes;
        }

        setValueEvent(cell) {
        	alert('click');
        }

        render() {
            return (
                <div className='game' >
                    {this._makeGame()}
                </div>
            );
        }
    }
  위의 소스는 현재 개발중인 sdoku_example프로젝트의 일부분을 수정한 부분이다. 해당 소스는 react와 box를 import를 해서 사용하도록 하고 있다. 그리고 페이지가 랜더링 하면서 box라는 JSX를 9개 생성하여 자식으로 만드는 과정을 담고 있다. 여기서 중요한것은 바로 `let boundClick = this.setValueEvent.bind(this);`와 `event={boundClick}` 부분이다. gameview class에서 setValueEvent함수를 생성하고 자식의 프로퍼티로 전달하는 과정인데. 보내기전에 미리 `.bind(this);` 처리하는것이 이벤트 전달의 핵심이다.  
  다음의 소스를 보자.
  
    import React, {Component} from 'react';
    import './index.less';

    export default class Cell extends Component{
        constructor(props) {
            super(props);
            this.state = {
                value: props.value,
                isFail: false
            };
        }

        getClassName() {
            var class_name = 'cell';
            if(this.state.isFail && !this.props.value) {
                class_name += ' fail';
            }

            return class_name;
        }

        clickItem(...args) {
            if(!this.props.value) {
                this.props.event(this, ...args);
            }
        }

        render() {
            return (
                <div className={this.getClassName()} onClick={this.clickItem.bind(this)}>
                    {this.state.value}
                </div>
            ); 
        }
    }
  
  GameView에서 만든 이벤트는 최종적으로 Cell의 프로퍼티로 들어 오는데 이때의 프로퍼티 명칭을 event로 주었다. onClick 이벤트가 발생하면 clickItem함수가 실행되도록 하였고 조건문을 통과하고 나면 `this.props.event(this, ...args)`를 수행하여 GameView에서 작성한 setValueEvent가 발생하도록 했다. 그런데 아까전에 `.bind(this);`가 중요하다고 했는데 그이유는 바로 this처리 때문이다. 주로 부모에서 자식의 이벤트가 감지가 필요한 경우의 대부분이 부모가 가지고 있는 변수나 상수를 참조해야하는 경우일것이다.(그런경우가 아니라면 굳이 부모에게 전달할 필요없이 자식이 처리하면 될테니...) 나의경우에는 전체적인 GameData관리를 부모가 하기 때문에 GameView가 Cell의 동작여부에 따라 그 값을 처리할 필요가 있었다. 그런데 단순히 함수만 프로퍼티로 전달해서 Cell에서 함수를 수행했더니 setValueEvent에서 접근하는 this가 GameView의 this가 아닌 Cell의 this에 접근을 하였다. 그래서 실질적으로 함수는 GameView에 있지만 GameView의 데이터가 전역변수가 아닌이상 접근할 방법이 없게 되어버렸다. 그렇다고 모든 자식에세 데이터 오브젝트를 전달할수도 없는 노릇이었다. 이때 활용된것이 바로 `.bind(this);` 이었다. 전달하기전에 미리 GameView의 this를 바인딩 시킨 후에 자식에세 전달하니 Cell에서 수행해도 정상적으로 GameView의 this영역에 접근할수 있었다.
  
  ##### 마무리
  음... 역시 난 아직 글을 참 못쓴다...  
  계속 쓰다보면 글쓰는 솜씨도 늘겠지...  
  늘었으면 좋겠다... 하하하. 이런글도 도움이 되는 누가 있었으면 좋겠다. ;)
  
[^1]: https://facebook.github.io/react/docs/events.html