---
layout: default
---

[The WET Manifesto](http://github.com/wetmanifesto/wetmanifesto.github.io): reasonable patterns for producing more expressive tests.  

<iframe src="https://ghbtns.com/github-btn.html?user=wetmanifesto&amp;repo=wetmanifesto.github.io&amp;type=watch&amp;count=true&amp;size=large"
  allowtransparency="true" frameborder="0" scrolling="0" width="170" height="30"></iframe><br/>

## The Horror

~~~java
@SuppressWarnings("unchecked")
@Test
  public void processEventShouldUpdateTheEventConsumerOffsetTable() {
    BSONTimestamp eventOffset = new BSONTimestamp(123, 456);
    EventConsumer<SomethingHappenedEvent> eventConsumer = mock(EventConsumer.class);
    doReturn(mongoConverter).when(subject.eventStreamMongoTemplate).getConverter();
    doReturn(fakeEvent).when(mongoConverter).read(SomethingHappenedEvent.class, jsonObject);
    doReturn(new ObjectId(EVENT_ID)).when(jsonObject).get("_id");
    doReturn(eventOffset).when(jsonObject).get("eventOffset");

    ArgumentCaptor<EventStreamOffset> captor = ArgumentCaptor.forClass(EventConsumerOffset.class);

    subject.processEvent(eventConsumer, SomethingHappenedEvent.class, json, consumerName);

    InOrder inOrder = Mockito.inOrder(eventConsumer, subject.eventConsumerOffsetRepository);
    inOrder.verify(eventConsumer).onEvent(fakeEvent);
    inOrder.verify(subject.eventStreamOffsetRepository).save(captor.capture());
    EventConsumerOffset eventConsumerOffset = captor.getValue();
    assertEquals(consumerName, eventConsumer.getEventConsumerName());
    assertEquals(eventOffset, eventConsumerOffset.getLastEventOffset());
}
~~~


## The Pillars of the WET Manifesto
__Readability over duplication__
*Instead of using private methods in your test to avoid code duplication, each unit test should take care of its own explicit setup, method calls and assertions. Unit tests should be isolated from each other so they can be understood without reading the entire test suite*

"DAMP and DRY are not contradictory, rather they balance two different aspects of a code's maintainability. Maintainable code (code that is easy to change) is the ultimate goal here...Tests often contain inherent duplication because they are testing the same thing over and over again, only with slightly different input values or setup code. However, unlike production code, this duplication is usually isolated only to the scenarios within a single test fixture/file. Because of this, the duplication is minimal and obvious, which means it poses less risk to the project than other types of duplication.

Furthermore, removing this kind of duplication reduces the readability of the tests. The details that were previously duplicated in each test are now hidden away in some new method or class. To get the full picture of the test, you now have to mentally put all these pieces back together.

Therefore, since test code duplication often carries less risk, and promotes readability, its easy to see how it is considered acceptable." - Chris Edwards (https://stackoverflow.com/questions/6453235/what-does-damp-not-dry-mean-when-talking-about-unit-tests)


__Single purpose for each test__
*Unit tests should fail for exactly one reason. The developer should be able to quickly read the test and know the single, exact defect in the production code that needs to be fixed. Strive for single assertions in your unit tests as this will emphasize the single functionality that you are testing*


__Tests should be well-organized__
*Follow the "given, when, then" structure when writing your tests, with clear and distinct logical breaks separating each block. Test setups should be kept to a minimal, without any extraneous data setup. Anyone reading the test should understand the purpose of each field you are creating in your setup data. Spacing is crucial for reading tests and help the eyes to logically parse tests.*

__Tests should have clear names__
*Test names should follow the syntax, spelling and grammar rules of your team’s primary language. Teams should norm on how a test name is constructed, but we recommend stating what happens when the method under test is called*

*For unit tests, the names should describe the method under test, the parameters passed in, and the assumption being made*

*For integration tests, the names should describe behavior instead of the specific parts of the production code that are being tested*

~~~java
def "add() returns the sum when the #{parameters_passed_in} are non-zero numbers"() {
    // test
}
~~~

~~~java
def "#add returns the sum when the #{parameters_passed_in} are non-zero numbers"() {
    // test
}
~~~

~~~java
def "add(1 + 1) returns the sum of 1 + 1"() {
    // test
}
~~~

~~~java
def "given two non-zero numbers, when add() is called then the sum of those numbers is returned"() {
    // test
}
~~~

<a href="https://github.com/wetmanifesto/wetmanifesto.github.io" class="github-corner"><svg width="80" height="80" viewBox="0 0 250 250" style="fill:#151513; color:#fff; position: absolute; top: 0; border: 0; right: 0;"><path d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"></path><path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2" fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path><path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z" fill="currentColor" class="octo-body"></path></svg></a><style>.github-corner:hover .octo-arm{animation:octocat-wave 560ms ease-in-out}@keyframes octocat-wave{0%,100%{transform:rotate(0)}20%,60%{transform:rotate(-25deg)}40%,80%{transform:rotate(10deg)}}@media (max-width:500px){.github-corner:hover .octo-arm{animation:none}.github-corner .octo-arm{animation:octocat-wave 560ms ease-in-out}}</style>
