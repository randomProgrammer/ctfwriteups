#Sharpturn#

##I think my SATA controller is dying.##
###HINT: git fsck -v###

We are given a git repo, clone it and run git fsck -v.

<pre><code>
Output:
Checking tree 2bd4c81f7261a60ecded9bae3027a46b9746fa4f
Checking commit 2e5d553f41522fc9036bacce1398c87c2483c2d5
error: sha1 mismatch 354ebf392533dce06174f9c8c093036c138935f3
error: 354ebf392533dce06174f9c8c093036c138935f3: object corrupt or missing
Checking commit 4a2f335e042db12cc32a684827c5c8f7c97fe60b
Checking tree 4c0555b27c05dbdf044598a0601e5c8e28319f67
Checking commit 7c9ba8a38ffe5ce6912c69e7171befc64da12d4c
Checking tree a1607d81984206648265fbd23a4af5e13b289f83
Checking tree cb6c9498d7f33305f32522f862bce592ca4becd5
Checking commit d57aaf773b1a8c8e79b6e515d3f92fc5cb332860
error: sha1 mismatch d961f81a588fcfd5e57bbea7e17ddae8a5e61333
error: d961f81a588fcfd5e57bbea7e17ddae8a5e61333: object corrupt or missing
Checking blob e5e5f63b462ec6012bc69dfa076fa7d92510f22f
Checking blob efda2f556de36b9e9e1d62417c5f282d8961e2f8
error: sha1 mismatch f8d0839dd728cb9a723e32058dcc386070d5e3b5
error: f8d0839dd728cb9a723e32058dcc386070d5e3b5: object corrupt or missing
</code></pre>

Hmm... 3 file corruption errors. Starting from the first:

<pre><code>
$ git cat-file -p 354ebf392533dce06174f9c8c093036c138935f3

#include <iostream>
#include <string>
#include <algorithm>

using namespace std;

int main(int argc, char **argv)
{
        (void)argc; (void)argv; //unused

        std::string part1;
        cout << "Part1: Enter flag:" << endl;
        cin >> part1;

        int64_t part2;
        cout << "Part2: Input 51337:" << endl;
        cin >> part2;

        std::string part3;
        cout << "Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194" << endl;
        cin >> part3;

        std::string part4;
        cout << "Part4: C.R.E.A.M. Get da _____: " << endl;
        cin >> part4;

        return 0;
}
</code></pre>

Okay, so we know his SATA controller is having problems, and the file is corrupt. Presumably there's some kind of bit flip here?
git checkout 2e5d --force (This commit is specified in fsck -v as the one where the corrupt file is).

<pre><code>
$ git hash-object sharp.cpp
> 8675cb049bbc546dee3233d4ada488f47b1f1ed3
Yep, definitely wrong! We're looking for 354ebf392533dce06174f9c8c093036c138935f3
</code></pre>

edit sharp.cpp 51337->31337

<pre><code>
$ git hash-object sharp.cpp
354ebf392533dce06174f9c8c093036c138935f3
</code></pre>

Bingo!

That's one down. Two to go.

<pre><code>
$ git cat-file -p d961f81a588fcfd5e57bbea7e17ddae8a5e61333
</code></pre>

outputs a modified sharp.cpp

<pre><code>
$ git diff 2e5d d57a
+       uint64_t first, second;
+       cout << "Part5: Input the two prime factors of the number 270031727027." << endl;
+       cin >> first;
+       cin >> second;
+
+       uint64_t factor1, factor2;
+       if (first < second)
+       {
+               factor1 = first;
+               factor2 = second;
+       }
+       else
+       {
+               factor1 = second;
+               factor2 = first;
+       }
+
</code></pre>

Check the prime factors of 270031727027 with wolfram alpha -> 29×271×1103×31151
Four...

<pre><code>
$git log
commit d57aaf773b1a8c8e79b6e515d3f92fc5cb332860
Author: sharpturn <csaw@isis.poly.edu>
Date:   Sat Sep 5 18:09:31 2015 -0700

    There's only two factors. Don't let your calculator lie.
</code></pre>
	
Okay, so this number is corrupted - should have only two factors.
I wrote a python script to flip a bit in the string "270031727027", check if it was a number and had two prime factors, and print the results. Hopefully there's only one!

<pre><code>
230031727027 [79, 2911794013L]
272031727027 [31357, 8675311L]
270231727027 [181, 1492992967L]
270131727027 [3, 90043909009L]
270021727027 [13, 20770902079L]
270033727027 [1409, 191649203L]
270031727827 [157, 1719947311L]
270031727127 [3, 90010575709L]
270031727067 [3, 90010575689L]
</code></pre>

Not one, but not too many. Switch em in for the original, check the hash!
<pre><code>
$ git hash-object sharp.cpp
42a825de0209c287c039b52370ac690a40d838c1
</code></pre>

Number two on the list (272031727027) does the job!

<pre><code>
$ git hash-object sharp.cpp
d961f81a588fcfd5e57bbea7e17ddae8a5e61333
</code></pre>

Last one.

<pre><code>
$ git cat-file -p f8d0839dd728cb9a723e32058dcc386070d5e3b5
</code></pre>
sharp.cpp again

<pre><code>
$ git diff d57a master

diff --git a/Makefile b/Makefile
new file mode 100644
index 0000000..e5e5f63
--- /dev/null
+++ b/Makefile
@@ -0,0 +1,6 @@
+
+CXXFLAGS:=-O2 -g -Wall -Wextra -Wshadow -std=c++11
+LDFLAGS:=-lcrypto
+
+ALL:
+       $(CXX) $(CXXFLAGS) $(LDFLAGS) -o sharp sharp.cpp
diff --git a/sharp.cpp b/sharp.cpp
index d961f81..f8d0839 100644
--- a/sharp.cpp
+++ b/sharp.cpp
@@ -2,8 +2,57 @@
 #include <string>
 #include <algorithm>

+#include <stdint.h>
+#include <stdio.h>
+#include <openssl/sha.h>
+
 using namespace std;

+std::string calculate_flag(
+               std::string &part1,
+               int64_t part2,
+               std::string &part4,
+               uint64_t factor1,
+               uint64_t factor2)
+{
+
+       std::transform(part1.begin(), part1.end(), part1.begin(), ::tolower);
+       std::transform(part4.begin(), part4.end(), part4.begin(), ::tolower);
+
+       SHA_CTX ctx;
+       SHA1_Init(&ctx);
+
+       unsigned int mod = factor1 % factor2;
+       for (unsigned int i = 0; i < mod; i+=2)
+       {
+               SHA1_Update(&ctx,
+                               reinterpret_cast<const unsigned char *>(part1.c_str()),
+                               part1.size());
+       }
+
+
+       while (part2-- > 0)
+       {
+               SHA1_Update(&ctx,
+                               reinterpret_cast<const unsigned char *>(part4.c_str()),
+                               part1.size());
+       }
+
+       unsigned char *hash = new unsigned char[SHA_DIGEST_LENGTH];
+       SHA1_Final(hash, &ctx);
+
+       std::string rv;
+       for (unsigned int i = 0; i < SHA_DIGEST_LENGTH; i++)
+       {
+               char *buf;
+               asprintf(&buf, "%02x", hash[i]);
+               rv += buf;
+               free(buf);
+       }
+
+       return rv;
+}
+
 int main(int argc, char **argv)
 {
        (void)argc; (void)argv; //unused
@@ -41,6 +90,11 @@ int main(int argc, char **argv)
                factor2 = first;
        }

+       std::string flag = calculate_flag(part1, part2, part4, factor1, factor2);
+       cout << "flag{";
+       cout << &lag;
+       cout << "}" << endl;
+
        return 0;
</code></pre>
		
Okay so added a Makefile, rounded out the flag generation code... which bit could be off? 

Notice the flag printing code: "cout << &lag;"
Doesn't look right! Try "cout << flag;".

<pre><code>
$ git hash-object sharp.cpp
f8d0839dd728cb9a723e32058dcc386070d5e3b5
</code></pre>

Another match!

compile the code, run it.

<pre><code>
>> Part1: Enter flag:
flag

>> Part2: Input 31337:
31337

>> Part3: Watch this: https://www.youtube.com/watch?v=PBwAxmrE194
okay...

>> Part4: C.R.E.A.M. Get da _____:
money

Part5: Input the two prime factors of the number 272031727027.
31357
8675311
</code></pre>

###>> flag{some hash here i didn't save}###







