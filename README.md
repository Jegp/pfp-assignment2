# Hand-in 2

Code is also available here: https://github.com/Jegp/pfp-assignment2

# Part 1
I solved this part by defining a function for distiguishing between characters
(a-z;A-A) and non-characters (everything else). With that I made a boolean
array of character (``T``) and non-character (``F``) elements. The indices
of that array can be used to create a list of the length of coherent pieces of
characters, which can finally be applied to ``partition`` the original text.

# Part 2

## 2.a
The exclusive scan was programmed by first recursing until the problem size
was ``<= 2``. For that size an array consisting of the first element and the
first element + the second element is defined. Throughout the recursion the
left result is carried on to the right side, so the scan can be complete on
the entire list.

## 2.b
For the unite and conquer part I used the code from the slides but extended it
to pad the array with zeroes, until the closest power of 2 of the array
length. I used zeroes because the would have no effect on the accumulator
function (``+``).

## 2.c
The following printout shows the estimated work and span for the divide and
conquer (first) and unite and conquer (second) algorithms:

    $> scan_divide([1, 2, 3, 4, 5, 6, 7, 8])
    [0, 1, 3, 6, 10, 15, 21, 28] : [int]
    [Work: 124, span: 81]

    $> scan_unite([1, 2, 3, 4, 5, 6, 7, 8])
    [0, 1, 3, 6, 10, 15, 21, 28] : [int]
    [Work: 165, span: 57]

The total amount of work is comparatively smaller in the first algorithm.
This is to be expected because the recursive algorithm resolves to fewer steps
than the unite and conquer implementation. However the span is higher because
my implementation of the divide and conquer version requires that the left
branches complete, before the right branches can be calculated.
A smarter version would make sure that the right branch could be
executed in parallel and summed afterwards.

# Part 3
I chose to solve this exercise using most significant digit (MSD) sort.
MSD sorting was implemented by first sorting the numbers into buckets of the
biggest existing base to avoid spending too much time on sorting bases that was
not present. The buckets were defined from 0 to the number of possible bases
(using base 10) that can be represented by a 32 bit integer. Later each base
position is sorted into buckets (using the ``radix_base_digit`` function) and
recursively sorted until there are no more positions left or the bucket is
empty.

This solution only supports postive numbers.

Below is the runtime of the radix sort implementation in ``part3.unesl``:

    $> radix_sort_msd([1, 2000, 4, 510, 20000030, 3, 2, 0, 2, 70123])
    [0, 1, 2, 2, 3, 4, 510, 2000, 70123, 20000030] : [int]
    [Work: 274, span: 49]

The span is significantly lower than the work in this case, proving that the
function is able to exploit at least some level of parallelism. The worst case
span depends on the largest base in the number, but is independent on the
length of the list.

# Part 4

## Part 4.a
Below is the runtime of the quicksort implementation

    $> quick_sort(["hi", "you", "there", "how", "are", "you"])
    [['a', 'r', 'e'], ['h', 'i'], ['h', 'o', 'w'], ['t', 'h', 'e', 'r', 'e'],
     ['y', 'o', 'u'], ['y', 'o', 'u']] : [[char]]
    [Work: 311, span: 145]

This implementation utilises a comparison function that compares two strings
``a`` and ``b``. It returns 1 if ``a`` is "larger" than ``b``, -1 if the
opposite is the case and 0 if they have the same ordering. Sorting picks
the first element of the list as the pivot, and recursively stitches the
elements together that are lesser (``lp``) and greater (``rp``).

## Part 4.b

Below is the runtime of the modified radix sort implementation

    $> radix_sort_string(["hi", "you", "there", "how", "are", "you"])
    [[['a', 'r', 'e']], [['h', 'i'], ['h', 'o', 'w'], ['t', 'h', 'e', 'r', 'e'],
     ['y', 'o', 'u'], ['y', 'o', 'u']]] : [[[char]]]
    [Work: 917, span: 215]

This algorithm is implemented by sorting each index of each entry at a time.
By recursively iterating through the indices of each word, until the end of
the words (or only one word is left), each number is sorted according to the
same principle as in part 3. The downside of this approach is that the
sorting of remaining indices is put off until the current index has been
sorted, since we cannot know which bucket the current index will be placed in.

## Part 4.c
The performance of the radix sort is significantly worse than the quicksort
implementation. This is probably caused by the fact that the radix sort has to
calculate the radix buckets of each character index, before the next index
can be processed.

# Appendix A: Part 1

    _AND(a, b) =
      if not(a) then F else b

    function _OR(a, b) =
      if a then T else b

    function _XOR(a, b) =
      if a then not(b) else T

    function ischar(c) =
      let gt_a = ord(c) >= ord('a');
          lt_z = ord(c) <= ord('z');
          gt_A = ord(c) >= ord('A');
          lt_Z = ord(c) <= ord('Z')
      in _OR(_AND(gt_a, lt_z), _AND(gt_A, lt_Z))

    function split_words(words) =
      let alpha_list = { ischar(x) : x in (words ++ " ") };
          idx_list = &(#words);
          alpha_idx = {i + 1: i in idx_list |
            _OR(i == #words - 1, _OR(not(alpha_list[i]),
                _AND(alpha_list[i], not(alpha_list[i + 1])))) };
          scatter_idx = {if x == 0 then alpha_idx[x]
                         else alpha_idx[x] - alpha_idx[x - 1]: x in &#alpha_idx};
          words_with_non_words = partition(words, scatter_idx)
      in {w: w in words_with_non_words | ischar(w[0])}

# Appendix B: Part 2

    -- Part a
    function scan_divide_rec(v, s, e, sum) =
      if e - s > 2 then let
        lp = scan_divide_rec(v, s, (s + e) / 2, sum);
        l_sum = lp[#lp - 1]
      in lp ++ scan_divide_rec(v, (s + e) / 2 + 1, e, l_sum)
      else [v[s] + sum, v[s] + v[e] + sum]

    function scan_divide(v) =
      if #v == 1 then [0]
      else let
        inc = scan_divide_rec(v, 0, #v - 1, 0)
      in [0] ++ {inc[i]: i in &(#inc - 1)}

    -- Part b
    function power_of_2(a, b) =
      if a <= b then b else power_of_2(a, b * 2)

    function pad_to_power_of_2(v) =
      let
        closest_power = power_of_2(#v, 2)
      in
        {if i < #v then v[i] else 0: i in &closest_power}

    function scan_unite(unpow_v) =   -- assumes #v is a power of 2
      if #unpow_v == 1 then [0]
      else let
        v = pad_to_power_of_2(unpow_v);
        vp = partition(v, {2: _ in &(#v / 2)});
        ps = {p[0]+p[1] : p in vp};
        sps = scan_unite(ps)
      in concat({[s, s+p[0]] : s in sps; p in vp})

# Appendix C: Part 3
    function _AND(a, b) =
      if not(a) then F else b

    def radix = 10

    function max_base(n, b) =
      if n < pow(radix, b) then b else max_base(n, b + 1)

    function radix_base_digit(n, d) =
      n / pow(radix, d) % 10

    def max_radix_i32 = max_base(pow(2, 32), 1)

    -- Sort a single bucket (list of numbers)
    function radix_sort_msd_bucket(v, r) =
      if #v <= 1 then v
      else let
        base_digits = {(radix_base_digit(n, r), n): n in v};
        buckets = scatter(base_digits, radix)
      in concat({if _AND(#b > 1, not(r < 1))
                 then radix_sort_msd_bucket(b, r - 1) else b: b in buckets})

    function radix_sort_msd(v) =
      let
        bases = {(max_base(n, 1), n): n in v};
        buckets = scatter(bases, max_radix_i32);
        sorted = {radix_sort_msd_bucket(buckets[i], i - 1): i in &max_radix_i32}
      in concat(sorted)
    function chr_cmp(a, b) =
      if ord(a) > ord(b) then 1
      else (if ord(a) < ord(b) then -1
      else 0)

    function str_cmp_rec(a, b, i) =
      if #a <= i then -1
      else if #b <= i then 1
      else let
        v = chr_cmp(a[i], b[i])
      in if v == 0 then str_cmp_rec(a, b, i + 1) else v

    -- Quicksort implementation
    function quick_sort_rec(v, p) =
      let
        split_groups = {(if str_cmp_rec(v[n + 1], p, 0) >= 0 then 1 else 0,
                         v[n + 1]): n in &(#v - 1)}; -- Exclude comparator
        xs = scatter(split_groups, 2);
        lp = if #xs[0] <= 1 then xs[0] else quick_sort_rec(xs[0], xs[0][0]);
        rp = if #xs[1] <= 1 then xs[1] else quick_sort_rec(xs[1], xs[1][0])
      in lp ++ [p] ++ rp

    function quick_sort(v) =
      if #v <= 1 then v
      else quick_sort_rec(v, v[0])

# Appendix D: Part 4
    -- Radix sort using strings
    function _AND(a, b) =
      if not(a) then F else b

    def radix = 10

    function max_base(n, b) =
      if n < pow(radix, b) then b else max_base(n, b + 1)

    function radix_base_digit(n, d) =
      n / pow(radix, d) % 10

    def max_radix_i32 = 2 -- ASCII have max 3 digits (0-indexed)

    --Sort a single bucket in a single position (vertically)
    function radix_sort_string_position(v, d, c) =
      if d < 0 then [v]
      else let
        base_digits = {(radix_base_digit(word[c], d), word): word in v};
        buckets = scatter(base_digits, radix)
      in buckets

    function radix_sort_string_bucket(v, d, c) =
      if d < 0 then [v]
      else let
        sorted_buckets = radix_sort_string_position(v, d, c)
      in {if #bucket > 1 then concat(radix_sort_string_bucket(bucket, d - 1, c))
         else bucket: bucket in sorted_buckets | #bucket > 0}

    function radix_sort_string_rec(v, c) =
      let
        char_buckets = radix_sort_string_bucket(v, max_radix_i32, c);
        sorted = {{w: w in bucket | #w == c + 1}: bucket in char_buckets};
        unsorted = {{w: w in bucket | #w > c + 1}: bucket in char_buckets};
        sorted_rec = {if #bucket > 1
          then concat(radix_sort_string_rec(bucket, c + 1))
          else bucket: bucket in unsorted}
      in sorted ++ unsorted

    function radix_sort_string(v) =
      let
        sorted_buckets = radix_sort_string_rec({{ord(x): x in word}: word in v}, 0)
      in {{{chr(x): x in word}: word in bucket}: bucket in sorted_buckets  | #bucket > 0}
