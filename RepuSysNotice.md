**Title: Reputation System Notied When Design**

## Suggested Edit And Edit

### Suggested Edit How to Work？

&emsp;功能的设计，如何保证每个人的修改都能存在，看见？并且能够回滚至起始点。(版本控制，修改队列，存储每个修改版本的内容，并可回滚) 。

[How does suggested edits work?](https://meta.stackexchange.com/questions/76251/how-do-suggested-edits-work)

&emsp;Suggested edits are held in a peer review queue of a fixed size. 比如 5。If the queue fills up, no more edit suggestions will be allowed until the queue has some empty space.

[Suggested Edits and Edit Review](https://stackoverflow.blog/2011/02/05/suggested-edits-and-edit-review/)

Suggested Edit Reasons below:

- To fix grammar and spelling mistakes
- To clarify the meaning of the post (without *changing* that meaning) 
- To include additional information only found in comments, so all of the information relevant to the post is contained in one place
- To correct minor mistakes or add updates as the post ages
- To add related resources or hyperlinks

### Suggested Edit 存在的问题

&emsp;**One of the problems** with the suggested edit system is that it takes almost zero time or effort to submit accept votes, or for that matter, reject votes. **It takes much longer to do the right thing** and improve bad suggestions.

**Possible solutions** (could be combined):

1. Don't act on accept/reject votes as soon as they're cast (or as soon as two agreeing votes are cast, in the case of SO). Force the suggestion to sit in the queue for n minutes.
   - If an improvement is submitted at any time, accept it immediately (like the way it works currently).
   - After the timer is up, if the total number of votes — both accept and reject — has reached some threshold, go with the majority.
   - Otherwise, add *m* more minutes to the timer and hope for more people to review the post or for someone to improve it.
2. If and when someone suggests improve, stop accepting accept/reject votes on that suggestion. Don't even show the suggestion in the queue anymore.
   - If the improvement is submitted, accept it immediately (like the way it works currently).
   - If the improvement is canceled, start showing it in the suggestion queue again.
   - After *p* minutes, if the improvement hasn't been submitted, assume the improver abandoned the attempt to fix the suggestion and start showing it in the suggestion queue again.

&emsp;**Another problem** with the current system is the lack of feedback for participants. Suggestion *reviewers* have no way to know whether the community agrees with their decisions, so bad reviewers keep on obliviously doing a bad job. **As long as reviewers are approving bad edits, users who suggest bad edits won't know that they need to get better**, let alone *want* to improve, and none of the rest of this post will matter. 说白了就是每个人都可以修改内容没错，但是有些较差的修改的人并不知道他的提交是没有用的，是无效的。这会导致他继续上传 较差的修改。

1. Give editors more feedback about whether their suggestions are good. In the case of rejected edits, be sure to explain why, perhaps with blue-level notifications.
2. Allow the community to mark low quality suggestion approvals, and give feedback to the editor and reviewer(s).
3. Make personal review stats more prominent in user profiles. This could be a hidden section, like the current helpful flags and votes sections.
4. Add an intermediate step between "suggest all the edits you want" and "you're banned from suggesting edits for a week" that somehow tells the user "a lot of your suggestions are getting rejected for reasons X, Y and Z; you might want to reread the edit guidelines."
5. Change the reviewer requirements from being based entirely on [the edit privilege](https://meta.stackoverflow.com/privileges/edit) to being based partially on whether the reviewer's previous accept/reject votes were correct. (Partially inspired by [this idea](https://meta.stackexchange.com/questions/96542/do-rejected-suggested-edits-affect-the-users-ability-to-make-future-suggested-ed).) This one went through numerous iterations in my head ~~and is probably too complicated to be implemented, but~~ maybe it'll help with brainstorming. Actually, tracking reviewer quality may not be as far-fetched as I thought! Some kind of "audit" system [is in the works](https://meta.stackexchange.com/a/143380/131713), per dev Kevin Montrose.

**但是关于这个讨论的一个决定：**(该讨论为：关于建议修改的结果是否应该显示?)

&emsp;[Decision on rejected edits should be displayed as a notification to the editor](https://meta.stackexchange.com/questions/120624/decision-on-rejected-edits-should-be-displayed-as-a-notification-to-the-editor)

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fkfiyggu1ij314o0hugq4.jpg)

参考： 

[What can we do to stop bad edits getting accepted?](https://meta.stackexchange.com/questions/137784/what-can-we-do-to-stop-bad-edits-getting-accepted)

[Add ability to Flag a User or Suggested Edit](https://meta.stackexchange.com/questions/101538/add-ability-to-flag-a-user-or-suggested-edit)

### Who can vote on a suggested edit?

- The owner of a post or a moderator may cast a binding vote to accept or reject any modification of their post. 

- All users with the [edit privilege](https://meta.stackoverflow.com/privileges/edit) may vote on suggested edits to posts.

- Users with [5000 rep](https://meta.stackoverflow.com/privileges/approve-tag-wiki-edits) may vote on suggested edits to tag wikis.

- **Two accept or reject votes** are required to remove the suggested edit from the queue and either apply the edit to the post or discard it. [It used to be a single vote (two on SO)](https://meta.stackexchange.com/a/151586/59303) and later [three on SO](https://meta.stackexchange.com/a/151586/59303)

  ​

### Revision History System

&emsp;Multiple edits made by the same person may be combined into a single revision, if they occur within a short period of time (currently 5 minutes). 

&emsp;Edits made by the original author are considered part of the base revision if submitted within 5 minutes of posting (unless someone else submits an edit first).

&emsp;It is possible to “roll back” changes made in a revision. This can be done during editing by selecting a previous revision to edit from the dropdown, but can also be accomplished via the "rollback" link displayed on non-current revisions within the revision history list - the latter is a little more convenient, and will earn you a bronze “Cleanup” badge when first used.

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fkf9iamzxej30m804y0t1.jpg)

### 如何处理多个用户同时修改

&emsp;When multiple editors submit changes, **the last one in “wins”, regardless of who began editing first.** Both revisions are preserved however, and changes lost can be restored either by rolling back to the previous revision or by manually copying text into a new revision.

&emsp;To promote good edits, a user who suggests an edit to someone else’s post will get +2 reputation points when that edit suggestion is approved. You do not get the reputation bonus if the edit is automatically approved, whether this is because you are editing your own post, you are editing a community wiki post, or you have at least 2000 reputation and all of your edits are auto-approved. Also, you are limited to a maximum of 1000 reputation points earned from edits.

### 必须记录每次用户提交的更改

&emsp; 防止发生纠纷时有据可循。可以根据每次提交的修改进行判断。

### 疑问 

1. 是否从一开始每个人都可以对问题或者回答进行编辑？（是）
2. **如何避免滥用编辑？**

&emsp;There are strict limits enforced. If a user (anonymous or registered) submits many rejected edits they will be automatically banned from suggesting edits. **The fixed size queue also helps protect us from abuse**.

&emsp;If a user without edit privileges proposes an edit that does not comply with the guidelines above, it is ordinarily rejected in the review process. Even if a bad edit is applied to a post, other users will generally fix it. Users with sufficient reputation may elect to roll back the post to a previous version (by viewing the revision history of the post and selecting the version they would like to display).

Additionally, any user who submits many rejected edits will be banned from suggesting further edits for 7 days.

> Any user can propose edits, but not all edits are publicly visible immediately. If a user has less than 2,000 reputation, the suggested edit is placed in a **review queue**. **Two accept or reject** votes are required to remove the suggested edit from the queue and either apply the edit to the post or discard it. Users with more than **2,000 reputation** are considered trusted community members and can edit posts without going through the review process.
>



详情可参考：

- [How does Editing Work](https://meta.stackexchange.com/questions/21788/how-does-editing-work)
- [Why can people edit my posts? How does editing work?](https://meta.stackexchange.com/help/editing)

## Reputation

### 疑问 

博问中每个用户的声望是从 1 开始，还是进行相关计算得出声望值？

**答：每个用户都是从1 开始。**

**既然每个人的声望从1 开始，会不会低于1 呢？**

&emsp;[Why does reputation start at 1, and have a lower bound of 1?](https://meta.stackexchange.com/questions/2621/why-does-reputation-start-at-1-and-have-a-lower-bound-of-1) 

不会低于1，但是如果有用户刚开始提了一个问题，被 vote down 了，那么他的声望还是1 ，不变吗？还是等到他获得其他声望后，减去这个vote down的？

答：还是1不会低于1 。这个减少声望的操作当只有1 声望时失效。



Stack Overflow 上专门一个界面来显示个人详细声望记录的界面 https://stackoverflow.com/reputation

参考 [How do I audit my reputation](https://meta.stackexchange.com/questions/43004/how-do-i-audit-my-reputation)

更加详细的声望增减规则[How does “Reputation” work?](https://meta.stackexchange.com/questions/7237/how-does-reputation-work)

## Flag

&emsp;Posts that receive enough spam or offensive flags from the community will be automatically deleted without moderator intervention.

### Spam

* Don't talk about your product / website / book / job too much.
* Don't tell - show!
* Don't include links except to support what you've written
  举个例子，Stack Overflow中对诸如如下回答的解决办法：
> 你可以参考这个链接：http://www.xxx.com/test . 

[Your answer is in another castle: when is an answer not an answer?](https://meta.stackexchange.com/questions/225370/your-answer-is-in-another-castle-when-is-an-answer-not-an-answer)

这张图片很形象。

> ![](https://ws4.sinaimg.cn/large/006tKfTcgy1fkd5p4myxrj30hs06jdl2.jpg) 

[图片来源](https://meta.stackexchange.com/questions/225370/your-answer-is-in-another-castle-when-is-an-answer-not-an-answer) 

### Rude or abusive

[Be Nice](https://stackoverflow.com/help/be-nice)

### Should be closed

### a duplicate

&emsp;This question has been asked before and already has an answer. If those answers do not fully address your question, please edit this question to explain how it is different or ask a new question.

### very low quality

### in need of moderator intervention

&emsp;审核人的主要作用应该就是 根据用户对每个Post（包括问题，回答，以及回复）的Flag进行追踪处理，而Flag 又是由用户进行标记的。主要处理的是这些标记 为 spam ，offensive , or need moderator notification 的。

A lot of the moderation work is mundane: deleting obvious spam, closing blatantly off-topic questions, and culling some of the worst-rated posts on the site. The ideal moderator does as little as possible, but those little actions may be powerful, visible, and highly concentrated.

**Once flagged, a post increments a flag count that shows up in the topbar for every moderator.**

### 疑问

给每个moderator 都发送通知吗？

> 另一个疑问：如果不是给每个人都发送通知，那么应该给具体哪一个moderator 发送通知？ 
> 如果给每个人发送通知，1. 那么如何防止多个moderator同时对post进行处理？
>
> 不影响，应该先给在线的Modertor 发送通知。

Stackoverflow上的Moderator也只有 24个 

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fkfjpapr4pj31kw0tmx0x.jpg)

### New Feature: Flag  a User

**&emsp;How about adding a "Flag this user for moderator attention" link to the user profile page, but only for users with zero questions and zero answers?**

## Moderator Privilege

*   Moderator votes are binding（审核人的投票具有约束力）. Any place we have voting — close, open, delete, undelete, offensive, migration, etc — that vote will reach the threshold and take effect immediately if a single moderator casts a vote.
*   Moderators can lock posts. Locked posts cannot be voted on or changed in any way.
*   Moderators can [protect questions](http://blog.stackoverflow.com/2010/06/new-protected-question-status/). Protected questions only allow answers by users with more than 10 reputation.
*   Moderators can see more data in the system, including vote statistics (but not ‘who voted for this post’) and user profile information.
*   Moderators can place users in [timed suspension](http://blog.stackoverflow.com/2009/04/a-day-in-the-penalty-box/), and delete users if necessary.
*   Moderators can perform large-scale maintenance actions such as merging questions and tags, tag synonym approvals, and so forth.

### Moderator Duty

1.  As a moderator, your actions now represent the community, so you will be held to a higher standard of behavior. You are an ambassador of trust, with the same sorts of rights that the official development team and community coordinators have.
2.  Your goal is to **guide the community with gentle — but firm — intervention**. Respect your fellow community members at all times; demonstrate fairness and impartiality in your actions.
3.  Whenever possible, **try to leave frequent comments on posts where you’ve taken (or considered taking) a moderator action**, explaining the reasoning. This is important so that community members can learn the norms of the community and the moderation policies.
4.  Keep the site reasonably on topic by **closing, migrating, or removing blatantly off-topic questions**.
5.  Regularly check for **flagged posts**, and decide if further action is warranted.
6.  In the case of serious disputes, **communicate directly with users via email** to help mediate and resolve those disputes.


### Moderator Constraint

1.  You must accept [the community moderator agreement](http://stackoverflow.com/legal/moderator-agreement) within 30 days of election or appointment and remain active on the site.
2.  You must stay in communication with your fellow moderators and work with them to resolve any disagreements within the team. [There is a process in place](http://meta.stackoverflow.com/questions/151606/handling-calls-to-remove-a-moderator/157258#157258) for a team to remove moderators who are unable or unwilling to cooperate.
3.  On Stack Overflow, due to its immense size and scale, there is another requirement. If you spend time on the site participating but aren’t regularly resolving flags, you may cede your right to remain a community moderator.


### Handling Moderator abused thier privileges

* [What recourse do I have if I believe a moderator has abused his/her privileges?](https://meta.stackexchange.com/questions/28867/what-recourse-do-i-have-if-i-believe-a-moderator-has-abused-his-her-privileges)
* [Handling Calls to Remove a Moderator](https://meta.stackexchange.com/questions/151606/handling-calls-to-remove-a-moderator/157258#157258)


### 如何成为Moderator

&emsp; 是达到一定声望自动成为 Moderator的吗？否 ，是来选举的（each site has moderators elected through popular vote. We hold regular elections to determine who these community moderators will be） 详情见 [Stack Overflow Moderator Voting ](https://stackoverflow.blog/2009/05/06/stack-overflow-moderator-voting-now-open/) 

> - To be eligible to vote, you must be a Stack Overflow user with **200 reputation**.
> - The list of candidates is shown in random order every time it is presented.
> - Only one vote can be cast per user. (no, it can’t be undone.)
> - After a week, the candidate with the most votes will be elected the new Stack Overflow moderator.
>
> 还有一个 关于 [Should Community Moderators be “elected for life”, or have terms?](https://meta.stackexchange.com/questions/984/should-community-moderators-be-elected-for-life-or-have-terms)

## Stack Overflow 大致运作流程

&emsp;以用户为主，通过`Reputation`来给予`Privilege`以及约束用户，主要由三种身份，第一个是 Stack Overflow 公司， 第二个是 Moderator 第三个就是普通用户( Moderator 也是从用户中评选出来的)。

你的声望越多，你所获得的特权就越大。

下面举个例子来解释：

> 一个用户发了一条垃圾问题信息，这时候，如果 Moderator看见了，可以直接把其删除，或者声望达到删除问题权限也可以进行删除。或者普通用户 Flag 该post 为 spam ，这时候每个Moderator都会收到一个通知，然后进行处理。
>
> 

Stack Overflow核心主要有以下几个系统构成，

* `Revision System`
* `Review Queue System`
* `Reputation System`
* `Badge System` 

其中`Revision System`,`Review Queue System` 这两个系统相对来说是网站给用户的特权，但是特权的约束又是由 `Reputation System` 是用来约束的。 这样就可以直接完全由用户为核心来运营这个站点，到如今，整个Stack Overflow 也只有24 个Moderator。最后 `Badge System`应该是算产品运营上的，用来提高以及调用用户使用站点积极性。这个点上博问中可以想点其他方案，但是最终效果是一样的。
![](https://ws3.sinaimg.cn/large/006tNc79gy1fkk88hjumnj30x00fgack.jpg)

PS：还有一个 分站点的思想。 每个分站点的风格都不同，但是制度都是大同小异。

### Badge System

The badge system exists for two reasons:

- to teach new users how Stack Exchange works
- to encourage activities that are positive to the community

[What are badges?](https://stackoverflow.com/help/what-are-badges)

### 关于如何处理用户大量提交无人回答的问题

&emsp; 博问中可以先不处理这种情况。[Asking Rate Limited](https://stackoverflow.com/help/asking-rate-limited) 

### 关于有人回答你的问题的处理办法

[What should I do when someone answers my question?](https://stackoverflow.com/help/someone-answers) 与博问中不同处在于 

> You may change which answer is accepted, or simply un-accept the answer, at any time.

**Please do not add a comment on your question or on an answer to say "Thank you".**

### How can a post be deleted?

[How does deleting work? What can cause a post to be deleted, and what does that actually mean? What are the criteria for deletion?](https://meta.stackexchange.com/questions/5221/how-does-deleting-work-what-can-cause-a-post-to-be-deleted-and-what-does-that) 

### About Bounty 赏金工作原理

[What is Bounty and how it works?](https://stackoverflow.com/help/privileges/set-bounties) 

[What is a bounty? How can I start one?](https://stackoverflow.com/help/bounty)



### Stack Overflow 上关于滥用投票的处理办法

[Why do I have a reputation change on my reputation page that says "voting corrected"?](https://stackoverflow.com/help/serial-voting-reversed)

两种情况： 一种是 两个用户间相互对其 问题或回答进行 投票赞同或反对。另一种是 一个用户对另一个不认识的用户的问题或回答进行投票或者反对。

Stack Overflow 后台有一个程序专门处理这种情况，每天进行一次检查。



关于 用户被删除后，其对他人投过的票的处理办法。

[Why do I have a reputation change on my reputation page that says 'User was removed'?](https://stackoverflow.com/help/user-was-removed)

> This message means that a user who voted for one of your posts had their account deleted (either by request or due to violating the network's terms of service). As a result, all of their votes were removed, and the reputation you gained or lost from them was undone. The resultant reputation change could be any amount; it could even be a reputation gain if enough of the removed votes were downvotes. All the reputation changes from a single user's deletion are rolled into a single event in the reputation page labelled "User was removed".
>
> This removal occurs whenever a user is deleted, unless that user had a very high reputation score. Because high-reputation users have usually cast a great many votes, removing all of them could be that much more disruptive to other users. In such cases, the staff use a special deletion that preserves the votes, resulting in no reputation change for those who had been voted on by that user.

### Stack Overflow 上对于 低质量的提问的处理

[Asking better question](https://stackoverflow.blog/2010/10/04/asking-better-questions/)

[Why are questions no longer being accepted from my account?](https://stackoverflow.com/help/question-bans)

### Stack Overflow 上对于Plagiarism(剽窃)抄袭类回答的处理

[How to reference material written by others](https://stackoverflow.com/help/referencing)

### 给问题／回答投赞同票的动机是什么？

&emsp;与之相对应的就是给回答投赞同票的动机？

> 给回答投赞同票的动机：
>
> - 一个问题可能会有多个回答，一个好的回答可能会被用户采纳为“最佳答案”并且获得 15 声望的奖励。这将鼓励用户提交更好的答案。
> - 当你的回答被选为最佳答案时，你会有一个心理：“感觉自己回答的不错。”

但是对于给问题投赞同来讲，

> - 很多人可能遇到最多的是“我也有这个问题，然后去看这个问题的回答？”，对于用户来讲可能是“在个人已经把我要问的给问了，我看看这个答案是什么？”关注点立马转向回答。
> - 缺少一个参照物，很多情况下你一次只点开一个问题，你眼前只有这个问题，没有可比对象，不像回答，可能有多个回答给你做参考。
> - 可以在点赞同票的位置上进行设计，比如放在问题的末尾。

## 博问中园豆和声望如何共存

&emsp; 园豆只仅仅用来提高问题关注度，与声望没有任何关联。园豆的数量与声望多少没有必然关系。

### 疑问

如何避免同个用户开两个账号进行刷声望行为？

**答：留一个板块显示 每天用户获取声望的排行榜**

## Privilege

### 关于 Vote up

&emsp;When you vote up, you are moving that content "up" so it will be seen by more people.

- By default, answers are sorted by number of votes.(Except for the accepted answer, which appears first unless it was written by the asker.)
- Upvotes on a question give the asker **+5** reputation.
- Upvotes on an answer give the answerer **+10** reputation.
- Upvotes on a comment help ensure the comment remains visible when there are many comments on a post, but do not give the author any reputation.
- **You can vote 30 times per UTC day, plus 10 more times on questions only.** 每天投票赞同的次数

### 关于Flag

**What happens when I flag something as *spam* or *abusive*?**

The spam and abusive flags are designed to automatically eliminate truly disruptive posts through the collaboration of the community.

- 3 flags -- post is banished from the front page. 
- 6 flags -- post is locked, deleted, and the owner loses 100 reputation.

需要有一个 **Review-Queue** 来处理这些Posts.

### Review-Queue-System is needed

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fkfltuqm4qj31kw1tztmp.jpg)

[When is a post removed from a review queue?](https://meta.stackexchange.com/questions/164288/when-is-a-post-removed-from-a-review-queue)

[Question flags, queues, edits, roomba, community♦, how does this actually work?](https://meta.stackexchange.com/questions/252048/question-flags-queues-edits-roomba-community-how-does-this-actually-work)

[What are the review queues, and how do they work?](https://meta.stackexchange.com/questions/161390/what-are-the-review-queues-and-how-do-they-work)





不同声望对应不同的权限，以及每个权限的解读参考

[Privilege](https://stackoverflow.com/help/privileges)

Stack Overflow 中的 **Review System** https://stackoverflow.com/review

主要有以下几类：

**Close Votes**

- Three "Leave Open" reviews
- One "Edit" review
- Post gets closed
- All close votes and recommend closure flags on the question either expire or are retracted

**Reopen Votes**

- Three "Leave Closed" reviews
- Post gets reopened
- All reopen votes on the question expire

**Suggested Edits**

- Two "Approve" reviews
- Two "Reject" reviews
- One review from the post author
- One "Improve Edit" review
- One "Reject and Edit" review
- [Automatic rejection due to edit conflict](https://meta.stackexchange.com/q/184992)
- The post is rolled back
- The tag no longer exists (tag wiki edits only)

**Low Quality Posts**

- Six "Recommend Deletion" reviews (for Answers) (four on Stack Overflow)
- One "Edit" review
- "Looks OK" reviews: the number of "very low quality" flags cast by users, plus a number set by the site (e.g. if the site ordinarily requires one review, but there are three "very low quality" flags, four "Looks OK" reviews are needed to remove it). [Source](https://meta.stackexchange.com/a/300659)
- The post gets closed
- The post is edited and the new revision passes the system's quality test

**First Posts**

- One action of any kind

**Late Answers**

- One action of any kind

**Triage** - [source](https://meta.stackexchange.com/a/300659)

- Three reviews of the same type

**Help & Improvement** - [source](https://meta.stackexchange.com/a/300659)

- One "Edit" action
- One "very low quality" action

## 声望系统帮助文档

&emsp;需要对声望系统的使用说明撰写一份帮助文档。

 Jeff Atwood 博客 https://stackoverflow.blog/authors/jeffatwood/page/8/

