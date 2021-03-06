# p5-hubot #

쉽게 확장 가능한 IRC bot hubot 을 소개하고 이를 사용, 확장하는 방법에 대해
이야기 합니다.

## p5? ##

Perl 5 를 뜻합니다. Perl 에서는 일반적으로 모듈이름이 대문자로
시작하지만, 소스저장소(repository) 이름으로는 다른 프로젝트와 구별하기
위해서 `p5` prefix 를 붙이곤 합니다.

## hubot? ##

![hubot](http://octodex.github.com/images/hubot.jpg)

[hubot](http://hubot.github.com/) 은 [github](https://github.com/)
에서 만든 채팅용 bot 입니다. adapter, script 를 확장해서 사용할 수
있습니다. CoffeeScript 로 쓰여져 있고, node.js 로 동작합니다.

                                            +---------+
                                            |  ascii  |
     +-----------+   +---------++----------+|---------|
     |           |   |         ||          ||  help   |
     | Campfire  | - |         ||          ||---------|
     |           |   |         || Brain    ||  roles  |
     +-----------+   |         || Robot    ||---------|
                     | Adapter || Listener ||  eval   |
                     |         || User     ||---------|
     +-----------+   |         || Message  ||  google |
     |           |   |         || Response ||---------|
     |   IRC     | - |         ||          || standup |
     |           |   |         ||          ||---------|
     +-----------+   +---------++----------+| twitter |
                                            |---------|
                                            |  .....  |
                                            +---------+

- script 를 확장해서 필요한 기능을 구현해서 사용할 수 있습니다
- 추가로 Adapter 를 구현해서 외부 서비스에 붙이면 이미 구현된 많은
  script 를 재사용할 수 있습니다.

### Adapter ###

`Hubot::Adapter` 는 연계될 서비스와의 인터페이스 입니다.

built-in Adapter 로는

- [Shell](http://search.cpan.org/~aanoaa/Hubot-0.0.9/lib/Hubot/Adapter/Shell.pm)
  - Script 개발과 디버깅에 적합합니다
  - input, output 을 Term 에서 주고 받습니다.
- [IRC](http://search.cpan.org/~aanoaa/Hubot-0.0.9/lib/Hubot/Adapter/Irc.pm)
- [Campfire](http://search.cpan.org/~aanoaa/Hubot-0.0.9/lib/Hubot/Adapter/Campfire.pm)

가 있습니다.

original 버전에는

- [Flowdock](https://github.com/github/hubot/wiki/Adapter:-Flowdock)
- [HipChat](https://github.com/github/hubot/wiki/Adapter:-HipChat)
- [Partychat](https://github.com/github/hubot/wiki/Adapter:-Partychat)
- [Talker](https://github.com/github/hubot/wiki/Adapter:-Talkerapp)
- [Tetalab](https://github.com/github/hubot/wiki/Adapter:-Tetalab)
- [Twilio](https://github.com/github/hubot/wiki/Adapter:-Twilio)
- [Twitter](https://github.com/github/hubot/wiki/Adapter:-Twitter)
- [XMPP](https://github.com/github/hubot/wiki/Adapter:-XMPP)
- [Gtalk](https://github.com/github/hubot/wiki/Adapter:-Gtalk)
- [Yammer](https://github.com/github/hubot/wiki/Adapter:-Yammer)
- [Skype](https://github.com/netpro2k/hubot-skype)
- [Jabbr](https://github.com/smoak/hubot-jabbr)

의 Adapter 가 구현되어 있습니다.

카카오톡, 마이피플등의 API 가 공개되어 있다면 확장해서 사용가능합니다.

### Scripts ###

특정 pattern 에 대해 반응하고 예약된 작업을 하는 논리적 코드의
집합입니다.

예를들면 adapter 로 부터 `hi` 라는 text input 이 들어왔을때 해야 할
작업(또는 처리)입니다.

    $robot->hear(qr/hi/i, sub { shift->send('hello') });

라는 코드가 있을때, `hi` 라는 텍스트가 정규표현식에 매칭되면, `hello`
라는 output 을 보내는 코드입니다.

    hshong> hi
    hubot> hello

이와 같이 필요로하는 다양한 상황에 대해 pattern 과 예약된 작업을
작성해서 확장해 나갈 수 있습니다.

이를 쉽게 하기 위해 편리한 인터페이스가 마련되어 있습니다.

## p5-hubot? ##

perl 로 만든
[hubot](http://search.cpan.org/~aanoaa/Hubot-0.0.7/lib/Hubot.pm)
입니다.

CoffeeScript 를 모르는 perl 사용자를 위해 만들어졌습니다.

### 확장 ###

`.pm` 파일을 만들고 PERL5LIB 환경변수에 `@INC` PATH 를 추가해서 사용할
수 있습니다.

`/path/to/lib/Hubot/Scripts/hello.pm`

    package Hubot::Scripts::hello;
    sub load {
        my ( $class, $robot ) = @_;
    
        ## robot respond only called its name first. `hubot xxx`
        $robot->respond(
            qr/hi/i,                 # aanoaa> hubot: hi
            sub {
                my $msg = shift;     # Hubot::Response
                $msg->reply('hi');   # hubot> aanoaa: hi
            }
        );
    
        $robot->hear(
            qr/(hello)/i,    # aanoaa> hello
                             # () 안에 있는건 capture 됨
                             # $msg->match->[0] eq 'hello'
            sub {
                my $msg = shift;
                $msg->send('hello');  # hubot> hello
            }
        );
    }

    1;
    
    =head1 SYNOPSIS

        hello - say hello
        hubot hi - say hi to sender

    =cut


`./hubot-scripts.json`

    ["hello","help"]

지금 작성한 스크립트를 사용할 수 있습니다.

### 주요 메소드 ###

- `load` 함수는 반드시 구현해 주어야 합니다. `hubot-scripts.json` 에
   포함된 스크립트를 메인 loop 실행전에 `load` 라는 이름의 함수로
   불러들이기 때문입니다.
   - `$class` 는 package 이름이고, `$robot` 은 `Hubot::Robot` 의
     instance 입니다.
- `$robot->respond(pattern, callback)`
  - `pattern` 에 대한 예약된 작업 `callback` 입니다.
  - `<robotname>[:,]?\s*<pattern>` 에 매치합니다.
  - `callback` 은 `Hubot::Response` 의 instance 를 argument 로
    가집니다.
    - `Hubot::Response` 는 `send`, `reply`, `random`, `finish`,
      `http` 등의 메소드를 가지는데 이를 이용하면 편리하게 adapter 에
      데이터를 보낼 수 있습니다.
- `$robot->hear(pattern, callback)`
  - `respond` 와 비슷하지만 `<pattern>` 에 매치합니다.
- `$robot->enter(callback)`
  - 채널에 누군가가 들어왔을때 실행됩니다.
- `$robot->leave(callback)`
  - 채널에 누군가가 나갔을때 실행됩니다.
- `$robot->catchAll(callback)`
  - 모든 이벤트에 대해 실행됩니다.

가장 좋은 방법은 이미 구현되어 있는 Script 의 소스코드를 살펴보는
것입니다.

[Hubot::Scripts::tell](https://metacpan.org/source/AANOAA/Hubot-Scripts-Bundle-0.0.12/lib/Hubot/Scripts/tell.pm)

### 설치 및 실행 ###

    $ cpanm Hubot
    $ export PERL5LIB=/path/to/lib:$PERL5LIB
    $ hubot
    hubot> hubot help
    # hubot help - Displays all of the help commands that Hubot knows about
    # hubot help <query> - Displays all help commands that match <query>
    # hello - say hello
    # hubot hi - say hi to sender
    Hubot> hello
    hello
    Hubot> hubot hi
    Shell: hi

    # irc 에 접속하시려면..
    $ hubot -a irc

## 더보기 ##

- http://hubot.github.com
- https://github.com/github/hubot
- https://github.com/github/hubot-scripts

## 결론 ##

`freenode #perl-kr` 채널에는 최신버전의 `p5-hubot` 이 `hongbot` 이라는
이름으로 동작하고 있습니다. 와서 구경해보시는 것도 좋겟지요. 확장해서
사용하고 싶은데 어려움을 느끼신다면 채널에 들어오셔서 저에게 메세지를
남겨주세요.

    ### @freenode #perl-kr
    you> hongbot tell hshong 아 이걸 이렇게 저렇게 하고 싶어여.

봇은 이제 그만 만들고 확장 script 를 만들어서 공유하면 좋겟습니다.
