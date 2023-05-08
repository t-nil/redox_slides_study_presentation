---
layout: intro 
---


# What is Ion Shell

- Running in Terminal
- Has own its scripting language 
- Is default shell in Redox Os
- Runs in Linux too

---

# Some Ion Shell code

```sh
echo $filename("/parent/filename.ext")
# Output: 
# filename

for data in @split("person, age, some data" ", ")
    echo $data
end
# Output: 
# person
# age
# some data

fn square x:int
    echo $(( x * x ))
end

let number = 3
square $number
# Output: 
# 9
square a
# Output: 
# ion: function argument has invalid type: expected int, found value 'a'
```

---

# Some parts of my contribution to Ion Shell

- Fixes in parser 
- Implement built in subst method
- Implement pipefail option

```sh{1|2|3-4|6|8|10|all}
let empty = []
let seq = @subst(  @empty [2 4])
# Output: 2 4
echo $seq

set -p -e

true | false

echo "should not print this"
```

---

# Why schemes in RedoxOs


Used for IPC, Inter Process Communication,  in RedoxOs
  
- Sharing resources
- Provides Services to programs

---

# Terms for schemes in RedoxOs

Terms to clarify

- [ ]  URL
- [ ]  Scheme 
- [ ]  Reference
- [ ]  Scheme Provider

---

# Scheme: What is an URL in RedoxOs

<div v-click class="text-xl p-4">

We are not talking about that kind of URL !

<br/>

<a>https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/What_is_a_URL</a>

<br/>

<hr>

</div>

<div v-click class="text-xl p-4">

We are talking about this kind of URL

<br/>  

file:/some_path/some_folder

<br/>  

<hr>

</div>

<div v-click class="text-xl p-4">

URL is the path to a resoruce in a scheme

</div>

---

# Scheme: What is an URL in RedoxOs

Terms to clarify

- [x]  URL
- [ ]  Scheme 
- [ ]  Reference
- [ ]  Scheme Provider

---

# Scheme: Components of an URL in RedoxOs

<br>

**URL** = \<scheme\>:\<reference\>

<br>

- **Scheme:**
  
  Name of scheme. Kind of the resource

<br>

- **Reference:**
  
  Pointer to a resource in a scheme

<br>

<hr>

<br>

Example: 

<br>

file:/some_path/some_folder 

---

# Terms for schemes in RedoxOs

Terms to clarify

- [x]  URL
- [x]  Scheme 
- [x]  Reference
- [ ]  Scheme Provider

---

# What are scheme providers 

Deamons which respond to requests for resources in scheme

|  Name    | Daemon  | Discription         |
|----------|---------|---------------------|
| ...      | ...     | ...                 |
| disk:    | ahcid   | Raw access to disks |
| file:    | redoxfs | Root filesystem     |
| orbital: | orbital | Windowing system    |
| ...      | ...     | ...                 |

---
layout: center
---

# Client and Server (Scheme Provider)

## Opening Resource

```mermaid
graph TD
   Program -->|1. forwards Open Request to| Kernel
   Kernel -->|2. forwards Request to| Scheme_Provider 
   Scheme_Provider -->|3. Create/Retrieve and start book keeping resource| Resource 
   Scheme_Provider -->|4. Forwards ID to| Kernel
   Kernel -->|5. Forwards ID to| Program
```

---
layout: center
---

# Reading from a resource

A request and response has the returned ID from opened resource

```mermaid
graph LR
   Program -->|1. Send request to| Kernel
   Kernel -->|2. Forwards request to| Scheme_Provider
   Scheme_Provider -->|3. Read| Resource 
   Scheme_Provider -->|4. Send back response to| Kernel
   Kernel -->|5. Forwards Response back to| Program
```

---

# Terms for schemes in RedoxOs

Terms to clarify

- [x]  URL
- [x]  Scheme 
- [x]  Reference
- [x]  Scheme Provider

---

# Register a scheme with Root Scheme

- Root scheme is empty String
- Reference after "**:**" is the name of the scheme to register

```rust{2|4}
fn main() {
    let mut scheme = VecScheme::new();

    let mut handler = File::create(":vec")
        .expect("Failed to create the vec scheme");
}
```

---

# Implementing a scheme provider

```rust
struct VecScheme {
    vec: Vec<u8>,
}

impl SchemeMut for VecScheme {
    fn open(&mut self, path: &str, _flags: usize, _uid: u32, _gid: u32) -> Result<usize> {
      // ...
    }
    
    fn read(&mut self, _id: usize, buf: &mut [u8]) -> Result<usize> {
      // ...
    }

    fn write(&mut self, _id: usize, buf: &[u8]) -> Result<usize> {
      // ...
    }
}
```

---

# Processing a request as scheme provider

```rust{8-10|12-13|15-16|all}
fn main() {
  // ... code from previous slide
  
  // Struct which contains data from request and for later response
  let mut packet = Packet::default();

  loop {
      // Wait for the kernel to send us requests
      let read_bytes = handler.read(&mut packet)
          .expect("vec: failed to read event from vec scheme handler");

      // calls certain function like read if client wants to read resources for example
      scheme.handle(&mut packet);

      handler.write(&packet)
          .expect("vec: failed to write response to vec scheme handler");
  }
}
```

---

# Intialize scheme provider

```rust{2-3|5-6|8-10|all}
fn main() {
    let mut vec_file = File::open("vec:/hi")
        .expect("Failed to open vec file");

    vec_file.write(b" Hello")
        .expect("Failed to write to vec:");

    let mut read_into = String::new();
    vec_file.read_to_string(&mut read_into)
        .expect("Failed to read from vec:");

    println!("{}", read_into); 
}
```
---

