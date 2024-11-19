# Thuật toán PAXOS

## Giới thiệu
- Trong một mạng, để đánh giá xem một process failure cần dựa trên kết quả đánh giá của đa số processes khác.
- Giao thức PAXOS ra đời để giải quyết vấn đề đánh giá này.
- Một process được coi là failure khi có nhiều hơn một nửa các processes nhận thấy rằng process đó đã failure (lý do có thể do mất kết nối, không running, etc)

## Các định nghĩa
- Có 3 roles mà các process có thể có: `proposer`,  `acceptor` và `learner`.
	- `proposer`: là process đề xuất các giá trị. Đa số thời gian, chỉ có một process đóng vai trò `proposer`, như vậy nó cũng được gọi là leader
	- `acceptor`: `acceptor` có thể accept các giá trị đề xuất. Một tập hợp các chấp nhận đại diện cho đa số được gọi là quorum. Khi một đề xuất đã được chấp nhận bởi quorum, chúng ta nói rằng đề xuất này đã được chọn. Có nghĩa là khi một đề xuất được accept bởi đa số các processes (hơn một nửa) thì đề xuất đó đã được chọn.
	- `learner`: sau khi đề xuất được chọn, `proposer` thông báo giá trị đến xuất đến cho các processes, các processes đó được gọi là `learner`.

## Giao thức
- Các giá trị đề xuất có nhãn duy nhất, nhãn là các số tự nhiên được sử dụng để sắp xếp các đề xuất.
- Giao thức paxos có 2 pha. Trong pha đầu, leader sẽ lắng nghe nếu có bất kỳ giá trị nào được đồng ý bởi các `acceptors`. Sau đó nó sẽ chọn một giá trị đó sẽ đảm bảo sự chính xác của thuật toán. Trong pha thứ hai, giá trị đề xuất được chọn từ giai đoạn 1 đến được đẩy đến cho `acceptor`. Sau khi một số lượng lớn các acceptor thừa nhận leader cho rằng giá trị đề xuất được chấp nhận, leader có thể thông báo cho `learner`.
  
- Cụ thể hơn, giao thức như sau:
- Pha 1, chuẩn bị:
	- `proposer`: Thứ nhất, `proposer` chọn một số duy nhất và quảng bá yêu cầu với nhãn `n` chuẩn bị cho `acceptors`.
	- `acceptor`: Khi một `acceptor` nhận một yêu cầu chuẩn bị với nhãn `n`, nó sẽ bỏ qua nếu nó đã nhận được yêu cầu chuẩn bị với nhãn cao hơn. Nếu không, nó sẽ không nhận bất kỳ yêu cầu nào có nhãn thấp hơn kể từ bây giờ (ngay sau khi nhận được nhãn). 
- Pha 2, đồng ý:
	- `proposer`: Nếu `proposer` nhận được phản hồi từ số quorum của `acceptors`, nó sẽ lựa chọn, trong số tất cả các đề nghị trả lại, đề nghị đó có nhãn cao nhất. `v` là giá trị của đề xuất đó. `Proposer` sau đó sẽ gửi yêu cầu chấp nhận cho `acceptor` có giá trị v, và nhãn n. nếu không nhận được trả lời từ đa số `acceptors`, nó sẽ quay lại pha 1.
	- `Acceptor`: Nếu một `acceptor` nhận được một yêu cầu chấp nhận có giá trị v và nhãn n, nó chấp nhận yêu cầu này, trừ khi nó đã đáp ứng một số yêu cầu chuẩn bị khác với nhãn cao hơn n. Nếu `acceptor` chấp nhận yêu cầu, nó sẽ gửi một phản hồi thông báo cho proposer. Nếu `proposer` nhận được một xác nhận quorum, thì bây giờ nó có thể quảng bá giá trị cho `learner`.

## When rounds fail

Các vòng có thể thất bại khi nhiều Proposer gửi các thông điệp Prepare mâu thuẫn, hoặc khi Proposer không nhận được quorum các phản hồi (Promise hoặc Accepted). Trong những trường hợp này, một vòng mới phải được bắt đầu với một số đề xuất cao hơn.

## Paxos có thể sử dụng để lựa chọn 1 leader
Lưu ý rằng trong Paxos, một Proposer có thể đề xuất "I am the leader" (hoặc ví dụ, "Proposer X is the leader"). Vì các đảm bảo về sự đồng thuận và tính hợp lệ của Paxos, nếu đề xuất này được một Quorum chấp nhận, thì Proposer đó sẽ được tất cả các nút còn lại biết đến như là một leader. Điều này đáp ứng yêu cầu của việc bầu chọn người lãnh đạo, vì luôn có một nút tin rằng mình là người lãnh đạo và một nút được biết đến là người lãnh đạo tại mọi thời điểm.

## Representation of the flow of messages in Paxos

### Basic Paxos without failures

In the diagram below, there is 1 Client, 1 Proposer, 3 Acceptors (i.e. the Quorum size is 3) and 2 Learners (represented by the 2 vertical lines). This diagram represents the case of a first round, which is successful (i.e. no process in the network fails).

![](../img/Basic_Paxos_without_failures.png)

Here, V is the last of (Va, Vb, Vc).

### Error cases in basic Paxos

The simplest error cases are the failure of an Acceptor (when a Quorum of Acceptors remains alive) and failure of a redundant Learner. In these cases, the protocol requires no "recovery" (i.e. it still succeeds): no additional rounds or messages are required, as shown below (in the next two diagrams/cases).

### Basic Paxos when an Acceptor fails

In the following diagram, one of the Acceptors in the Quorum fails, so the Quorum size becomes 2. In this case, the Basic Paxos protocol still succeeds.

```
Client   Proposer      Acceptor     Learner
   |         |          |  |  |       |  |
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(1)
   |         |          |  |  !       |  |  !! FAIL !!
   |         |<---------X--X          |  |  Promise(1,{Va, Vb, null})
   |         X--------->|->|          |  |  Accept!(1,V)
   |         |<---------X--X--------->|->|  Accepted(1,V)
   |<---------------------------------X--X  Response
   |         |          |  |          |  |
```

### Basic Paxos when a redundant learner fails

In the following case, one of the (redundant) Learners fails, but the Basic Paxos protocol still succeeds.

```
Client Proposer         Acceptor     Learner
   |         |          |  |  |       |  |
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(1)
   |         |<---------X--X--X       |  |  Promise(1,{Va,Vb,Vc})
   |         X--------->|->|->|       |  |  Accept!(1,V)
   |         |<---------X--X--X------>|->|  Accepted(1,V)
   |         |          |  |  |       |  !  !! FAIL !!
   |<---------------------------------X     Response
   |         |          |  |  |       |
```

### Basic Paxos when a Proposer fails

In this case, a Proposer fails after proposing a value, but before the agreement is reached. Specifically, it fails in the middle of the Accept message, so only one Acceptor of the Quorum receives the value. Meanwhile, a new Leader (a Proposer) is elected (but this is not shown in detail). Note that there are 2 rounds in this case (rounds proceed vertically, from the top to the bottom).

```
Client  Proposer        Acceptor     Learner
   |      |             |  |  |       |  |
   X----->|             |  |  |       |  |  Request
   |      X------------>|->|->|       |  |  Prepare(1)
   |      |<------------X--X--X       |  |  Promise(1,{Va, Vb, Vc})
   |      |             |  |  |       |  |
   |      |             |  |  |       |  |  !! Leader fails during broadcast !!
   |      X------------>|  |  |       |  |  Accept!(1,V)
   |      !             |  |  |       |  |
   |         |          |  |  |       |  |  !! NEW LEADER !!
   |         X--------->|->|->|       |  |  Prepare(2)
   |         |<---------X--X--X       |  |  Promise(2,{V, null, null})
   |         X--------->|->|->|       |  |  Accept!(2,V)
   |         |<---------X--X--X------>|->|  Accepted(2,V)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |
```

### Basic Paxos when multiple Proposers conflict

The most complex case is when multiple Proposers believe themselves to be Leaders. For instance, the current leader may fail and later recover, but the other Proposers have already re-selected a new leader. The recovered leader has not learned this yet and attempts to begin one round in conflict with the current leader. In the diagram below, 4 unsuccessful rounds are shown, but there could be more (as suggested at the bottom of the diagram).

```
Client   Proposer       Acceptor     Learner
   |      |             |  |  |       |  |
   X----->|             |  |  |       |  |  Request
   |      X------------>|->|->|       |  |  Prepare(1)
   |      |<------------X--X--X       |  |  Promise(1,{null,null,null})
   |      !             |  |  |       |  |  !! LEADER FAILS
   |         |          |  |  |       |  |  !! NEW LEADER (knows last number was 1)
   |         X--------->|->|->|       |  |  Prepare(2)
   |         |<---------X--X--X       |  |  Promise(2,{null,null,null})
   |      |  |          |  |  |       |  |  !! OLD LEADER recovers
   |      |  |          |  |  |       |  |  !! OLD LEADER tries 2, denied
   |      X------------>|->|->|       |  |  Prepare(2)
   |      |<------------X--X--X       |  |  Nack(2)
   |      |  |          |  |  |       |  |  !! OLD LEADER tries 3
   |      X------------>|->|->|       |  |  Prepare(3)
   |      |<------------X--X--X       |  |  Promise(3,{null,null,null})
   |      |  |          |  |  |       |  |  !! NEW LEADER proposes, denied
   |      |  X--------->|->|->|       |  |  Accept!(2,Va)
   |      |  |<---------X--X--X       |  |  Nack(3)
   |      |  |          |  |  |       |  |  !! NEW LEADER tries 4
   |      |  X--------->|->|->|       |  |  Prepare(4)
   |      |  |<---------X--X--X       |  |  Promise(4,{null,null,null})
   |      |  |          |  |  |       |  |  !! OLD LEADER proposes, denied
   |      X------------>|->|->|       |  |  Accept!(3,Vb)
   |      |<------------X--X--X       |  |  Nack(4)
   |      |  |          |  |  |       |  |  ... and so on ...
```

### Basic Paxos where an Acceptor accepts Two Different Values

In the following case, one Proposer achieves acceptance of value V1 by one Acceptor before failing. A new Proposer prepares the Acceptors that never accepted V1, allowing it to propose V2. Then V2 is accepted by all Acceptors, including the one that initially accepted V1.

```
Proposer    Acceptor     Learner
 |  |       |  |  |       |  |
 X--------->|->|->|       |  |  Prepare(1)
 |<---------X--X--X       |  |  Promise(1,{null,null,null})
 x--------->|  |  |       |  |  Accept!(1,V1)
 |  |       X------------>|->|  Accepted(1,V1)
 !  |       |  |  |       |  |  !! FAIL !!
    |       |  |  |       |  |
    X--------->|->|       |  |  Prepare(2)
    |<---------X--X       |  |  Promise(2,{null,null})
    X------>|->|->|       |  |  Accept!(2,V2)
    |<------X--X--X------>|->|  Accepted(2,V2)
    |       |  |  |       |  |
```

### Basic Paxos where a multi-identifier majority is insufficient

In the following case, one Proposer achieves acceptance of value V1 of one Acceptor before failing. A new Proposer prepares the Acceptors that never accepted V1, allowing it to propose V2. This Proposer is able to get one Acceptor to accept V2 before failing. A new Proposer finds a majority that includes the Acceptor that has accepted V1, and must propose it. The Proposer manages to get two Acceptors to accept it before failing. At this point, three Acceptors have accepted V1, but not for the same identifier. Finally, a new Proposer prepares the majority that has not seen the largest accepted identifier. The value associated with the largest identifier in that majority is V2, so it must propose it. This Proposer then gets all Acceptors to accept V2, achieving consensus.

```
Proposer           Acceptor        Learner
 |  |  |  |       |  |  |  |  |       |  |
 X--------------->|->|->|->|->|       |  |  Prepare(1)
 |<---------------X--X--X--X--X       |  |  Promise(1,{null,null,null,null,null})
 x--------------->|  |  |  |  |       |  |  Accept!(1,V1)
 |  |  |  |       X------------------>|->|  Accepted(1,V1)
 !  |  |  |       |  |  |  |  |       |  |  !! FAIL !!
    |  |  |       |  |  |  |  |       |  |
    X--------------->|->|->|->|       |  |  Prepare(2)
    |<---------------X--X--X--X       |  |  Promise(2,{null,null,null,null})
    X--------------->|  |  |  |       |  |  Accept!(2,V2)
    |  |  |       |  X--------------->|->|  Accepted(2,V2)
    !  |  |       |  |  |  |  |       |  |  !! FAIL !!
       |  |       |  |  |  |  |       |  | 
       X--------->|---->|->|->|       |  |  Prepare(3)
       |<---------X-----X--X--X       |  |  Promise(3,{V1,null,null,null})
       X--------------->|->|  |       |  |  Accept!(3,V1)
       |  |       |  |  X--X--------->|->|  Accepted(3,V1)
       !  |       |  |  |  |  |       |  |  !! FAIL !!
          |       |  |  |  |  |       |  |
          X------>|->|------->|       |  |  Prepare(4)
          |<------X--X--|--|--X       |  |  Promise(4,{V1(1),V2(2),null})
          X------>|->|->|->|->|       |  |  Accept!(4,V2)
          |       X--X--X--X--X------>|->|  Accepted(4,V2)
```

### Basic Paxos where new Proposers cannot change an existing consensus

In the following case, one Proposer achieves acceptance of value V1 of two Acceptors before failing. A new Proposer may start another round, but it is now impossible for that proposer to prepare a majority that doesn't include at least one Acceptor that has accepted V1. As such, even though the Proposer doesn't see the existing consensus, the Proposer's only option is to propose the value already agreed upon. New Proposers can continually increase the identifier to restart the process, but the consensus can never be changed.

```
Proposer    Acceptor     Learner
 |  |       |  |  |       |  |
 X--------->|->|->|       |  |  Prepare(1)
 |<---------X--X--X       |  |  Promise(1,{null,null,null})
 x--------->|->|  |       |  |  Accept!(1,V1)
 |  |       X--X--------->|->|  Accepted(1,V1)
 !  |       |  |  |       |  |  !! FAIL !!
    |       |  |  |       |  |
    X--------->|->|       |  |  Prepare(2)
    |<---------X--X       |  |  Promise(2,{V1,null})
    X------>|->|->|       |  |  Accept!(2,V1)
    |<------X--X--X------>|->|  Accepted(2,V1)
    |       |  |  |       |  |
```

## Multi-Paxos (later)
## Cheap Paxos (later)
## Fast Paxos (later)
## Generalized Paxos (later)
## Byzantine Paxos (later)