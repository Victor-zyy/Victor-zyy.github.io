+++
title = "cs144-lab1"
author = ["zyy"]
date = 2025-03-22T19:59:00+08:00
tags = ["cs144", "c++"]
categories = ["net"]
draft = false
showtoc = true
+++

## Overview {#overview}


### Course Learn {#course-learn}

operations idempotency meaning:

Idempotency is a property of certain operations or API requests that ensures performing the operation multiple times yields the same result as if it were executed only once. (GET PUT)
And the result we expect is **result form** not **result value**.

A non-idempotent operation, on the other hand, changes the state with each execution, meaning that subsequent calls with the same input can yield different results. This is crucial to recognize because it can lead to unintended side effects when such operations are repeated. (POST)

short packets data can be

-   lost
-   reorder
-   alter
-   duplicate


### line of code {#line-of-code}

From the startup code, i used almost 122 lines of code to design ByteStream in lab0 which is larger than  TA in class - 82 lines of code.

how to run this scripts ?

```sh
$ sudo apt install slocount
$ ./scripts/lines-of-code ; in mindow directory
             ByteStream:   122 lines of code
             Reassembler:   26 lines of code
$
             ByteStream:    82 lines of code
             Reassembler:   26 lines of code
```


### TCP Receiver {#tcp-receiver}

In this assignment , what you're gonna do is to rearrage the bytestream form unreliable to reliable and then to offer to the Socket that can be read by the web-application like webget, etc.

why do this ?

TCP robustness against reordering and duplication comes form its ability to stitch arbitrary excerpts of the byte stream back into the original stream. Implementing this in a discrete testable module will make handling incoming segments easier.


#### Incoming Substring {#incoming-substring}

```nil

normal format

"abcdefghaaa"

{index: 0, "abcd"}
{index: 4, "efg"}
{index: 7, "ha"}
{index: 9, "aa", last_string: ture}

or

"zyyll"
{index: 0, "z"}
{index: 0, "zyy"}
{index: 3, "ll", last_string: true}

.etc.
```

your Reassember module should handle many corner cases like lost, duplicate, altered, and reordered.


## Outline of Reassember Class {#outline-of-reassember-class}

The whole incoming substring construct format is like this.
![](/cs144/images/lab1_reassember.png)


### code procedure {#code-procedure}


#### step 1. check whether the incoming substring can be put into Bytestream or not? {#step-1-dot-check-whether-the-incoming-substring-can-be-put-into-bytestream-or-not}

From the picture, we know that the incoming string's index should not lie beyond first unaccepted index.
And, similarly an duplicated substring which doesn't exceed the first_unassembled_index is not allowed to put into the bytestream module. So the conditions are like this below.

```c++
// Step1. discard message for duplicate and lie beyond size
uint64_t avi = this->writer().available_capacity();
uint64_t pos = 0;
uint64_t cnt = 0;
if(first_index > next_index + avi)
  return; // lie beyond
if(first_index + data.size() <= next_index){
  if(is_last_substring) // corner case like emtpy string "" and close
    this->output_.writer().close();
  return; // duplicate
}
if(first_index + data.size() > next_index + avi){
  // not fully lie beyond but we need to doc the state of temp
  // update the beyond_
  if(is_last_substring){
    // need to check
    beyond_index = first_index + data.size();
    nclose_state = true;
  }
  // |-----|
  //    |-----|
  // [pos, cnt]
  cnt = first_index + data.size() - next_index -avi;
  pos = data.size() - cnt;
  data.erase(pos, cnt);
}
```


#### step 2. substrings can be put or assembled {#step-2-dot-substrings-can-be-put-or-assembled}

the incoming substring can be like in the picture below.
![](/cs144/images/lab1_string_handle.png)

if the substring covers the next_index,  we have to put into the Bytestream, if not we have to put into the reassember internal storage. To simplify the reassembler class, i chose **std::unordered_map&lt;uint64_t, std::stirng&gt;** as the internal storage for reassember, that's because we can find and iterate it fast.

```c++

  // two conditions like 1). repeat send but large enough , then need to be merged
  // 2). not sequenced needed to be put in map_ , then need to be merged
  if(first_index <= next_index){
    // |-----|
    //    |-----|
    pos = 0;
    cnt = next_index - first_index;
    data.erase(pos, cnt);
    this->output_.writer().push(data);
    // update next_index;
    next_index += data.size();
    // then check if need to merge and push
    auto it = find_next_assembly(next_index);
    while(it != map_.end() && map_.size() > 0){
      pos = 0;
      cnt = next_index - it->first;
      it->second.erase(pos, cnt);
      this->output_.writer().push(it->second);
      next_index += it->second.size();
      map_.erase(it);
      it = find_next_assembly(next_index);
    }
    // |-----| pushed
    //   |---| pending
    // do some clear
    this->unassmbly_clear(next_index);
}
```

When I put the substring in the bytestream module, we have to think over whether there are other unassembled strings can be merged. The function **find_next_assembly** is the way to find one, I choose while loop structure to merge that.

Last but not least, I need do some clear in case that while-loop finds nothing to merge but need to discard like

```c++
// |-----| pushed
//   |---| pending
// do some clear
```


#### step3, string that put into the internal storage of reassembler {#step3-string-that-put-into-the-internal-storage-of-reassembler}

Well, strings that put into the map\_ might also be merged in some cases, like the string cover the one in map_, or the string can splice with another one in the map\_. there are many cases we can think of, but only two conditions can handle these cases. So we need to check which one it is , and if not just insert into the internal storage.

Here are the two conditions.
![](/cs144/images/lab1_conditon_merge.png)

And the code implementation is just like do the left check loop until merged, after that do the same thing as the right loop check. And if the program runs out of the two loop, we can figure it out that this string need not be merged and just put into the map\_.

```c++
void Reassembler::merge_unassembly_map(uint64_t index, std::string data){
  // Condition 1. find left
  auto it_left = find_merge_left(map_, index, data);
  uint64_t cnt = 0;
  uint64_t pos = 0;
  uint64_t i = 0;
  while(it_left != map_.end()){
    if(index + data.size() >= it_left->first + it_left->second.size()){
      map_.erase(it_left);
      map_.insert({index, data});
    }else{
      if(index == it_left->first){
        //do nothing
        break;
      }else{
        cnt =  it_left->first + it_left->second.size() - index - data.size();
        pos = it_left->second.size() - cnt;
        for(i = 0; i < cnt; i++)
          data.push_back(it_left->second[pos + i]);
        map_.erase(it_left);
        map_.insert({index, data});
      }
    }
    it_left = find_merge_left(map_, index, data);
  }

  auto it_right = find_merge_right(map_, index, data);
  while(it_right != map_.end()){
    if(index + data.size() <= it_right->first + it_right->second.size()){
      // do nothing
      return;
    }else{
      cnt = index + data.size() - it_right->first - it_right->second.size();
      pos = data.size() - cnt;
      for(i = 0; i < cnt; i++){
        it_right->second.push_back(data[pos+i]);
      }
      // to avoid the index is in the map
      if(index != it_right->first){
         it_right = map_.find(index);
         if(it_right != map_.end())
           map_.erase(it_right);
      }
      return;
    }
    it_right = find_merge_right(map_, index, data);
  }
  //
  map_.insert({index, data});
  return;
}
```


### corner case {#corner-case}

"bz", index 1, is last string ture; capacity = 1

this case demostrates that, the last string has come first. But we don't have too much room to put into the internal storage. So we need to do some check and handle this case. In my mind, I choose to document the **beyond_index** it finally reaches. So when everythings works fine, i have to check if the beyond_index equals the next_index or not in the last.

```c++

if(is_last_substring){
  close_state = true;
}

if(!nclose_state){
  beyond_index = next_index;
}

if(close_state && map_.size() == 0 && beyond_index == next_index){
  this->output_.writer().close();
}

return;
```


### line of code {#line-of-code}

```sh
./scripts/lines-of-code
             ByteStream:   122 lines of code
             Reassembler:  176 lines of code
```

Lastly, i passed the tests, but the codes i wrote are a little bit clumsy. I will implement and upgrade this class module in the future to make it easy to use.

{{< figure src="/cs144/images/lab1_successfuly.png" >}}
