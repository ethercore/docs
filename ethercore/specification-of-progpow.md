# Specification of ProgPoW

ProgPoW is a new Proof-of-Work algorithm which is an extension of Ethash to combat specialized hardwares for mining cryptocurrency. Although Ethash was considered to prevent ASIC \(Application Specific Integrated Circuit\) chips from mining cryptocurrencies, several exploits from algorithm allowed chip designing companies and some bigger miners to take over the network. To defeat the centralization of network and to make the network more secure, the modification of algorithm was needed and therefore we suggest ProgPoW as a successor of Ethash algorithm.

To be specific for our goal for developing ProgPoW, ever since the first Bitcoin mining ASIC was released, many new Proof-of-Work algorithms have been created with the intention of being “ASIC-resistant”. The goal of “ASIC-resistance” is to resist the centralization of PoW mining power such that these coins couldn’t be so easily manipulated by a few mining companies.

The development goal of ProgPoW is to utilize the algorithm’s use for what is available on commodity hardwares where every miner can buy worldwide. To fulfill it’s target, ProgPoW will use every computing resource available on graphic card. Compared with this, if the algorithm is to be implemented on a specialized hardwares like customized FPGA cards or ASIC boards, there should be little opportunity for efficiency gains compared to a commodity graphic cards.

In many years of experience in developing a proof-of-work algorithm for cryptocurrency algorithms, ProgPoW is a superior algorithm for combating specialized mining devices. There are many PoW currencies that exist, including Bitcoin, Litecoin, Ethereum, Zcash, and much more, and they are using many different proof-of-work algorithms for computing hashes like SHA256D, Dagger-Hashimoto, Yescrypt, Aragon2D, Lyra2v, etc. When we choose hashing algorithms among them, we look for “ASIC resistance”, or “FPGA resistance”.

By following the design, ProgPoW can be considered as a special extension or derivative algorithm from Dagger-Hashimoto, may parts of the code are inherited from the original algorithm and it actually relies on to make the algorithm memory intensive and thus to be ASIC resistant. While the inherited part of the algorithm will utilize the DRAM part of the graphic card, it is possible for ProgPoW’s parameter calculated by number of blocks can utilize the core of GPU. Like any other core intensive algorithm, ProgPoW will utilize the computation of graphic cards to maximize the efficiency of blocking the development of specialized hardwares.

Therefore, the major modifications from Ethash will be the following: • Increases mix state. • Increases the DRAM read from 128 bytes to 256 bytes. • Changes Keccak\_f1600 \(with 64-bit words\) to Keccak\_f800 \(with 32-bit words\) to reduce impact on total power • Adds a random sequence of math in the main loop. • Adds reads from a small, low-latency cache that supports random addresses.

While a custom ASIC to implement this algorithm is still possible, the efficiency gains available are minimal. The majority of a commodity GPU is required to support the above elements. Our design goal of ProgPoW is to have about 1.1 ~ 1.2x efficiency gains if specialized hardwares were designed for ProgPoW while this is much less gain expected from 2x for Ethash algorithm or 100x gain for Equihash ASIC miners.

## Reasons to design Commodity hardware friendly Proof-of-Work algorithm

According to the recent report, more than half of the computing power for Bitcoin network is coming from a single province area in China. If the local provider fails to provide their service, Bitcoin may face uncomfortable delays for processing payments online. Not only the downtime, but also there are a serious risk of network centralization which exposes Bitcoin to the risk of third party interference and censorship, etc.

With the growth of large mining pools, the control of hashing power has been delegated to the top few pools to provide a steadier economic return for small miners. While some have made the argument that large centralized pools defeats the purpose of “ASIC resistance,” it’s important to note that ASIC based coins are even more centralized for several reasons. 1. High barrier to entry: ASIC miners are rich enough to invest capital and ecological resources on a new coin miner chip. Thus, initial coin distribution through ASIC mining will be very limited causing centralized economic bias for the coin ecosystem. 2. Delegated centralization vs implementation centralization: While mining pool centralization is delegated, hardware monoculture is not: only the limited buyers of this hardware can participate so there isn’t even the possibility of divesting control on short notice. 3. No natural distribution: There isn’t an economic purpose for ultra-specialized hardware outside of mining and thus no reason for most people to have it. 4. No reserve group: Thus, there’s no reserve pool of hardware or reserve pool of interested parties to jump in when coin price is volatile and attractive for manipulation. 5. No obvious decentralization of control even with decentralized mining: Once large custom ASIC makers get into the game, designing back-doored hardware is trivial. ASIC makers have no incentive to be transparent or fair in market participation.

While some coins and Blockchains may target individual miners for the concept of “ASIC resistance”, the entire idea may be a fallacy. CPUs and GPUs themselves are ASICs. Any algorithm that can run on a commodity ASIC \(CPU or GPU\) by definition can have a customized ASIC created for it with slightly less functionality. Some algorithms are intentionally made to be “ASIC friendly” - where an ASIC implementation is drastically more efficient than the same algorithm running on general purpose hardware. The protection that this offers when the coin is unknown also makes it an attractive target for a dedicated mining ASIC company as soon as it becomes useful.

Therefore, ASIC resistance is: the efficiency difference of specialized hardware versus hardware that has a wider adoption and applicability. A smaller efficiency difference between custom vs general hardware mean higher resistance and a better algorithm. This efficiency difference is the proper metric to use when comparing the quality of PoW algorithms. Efficiency could mean absolute performance, performance per watt, or performance per dollar - they are all highly correlated. If a single entity creates and controls an ASIC that is drastically more efficient, they can gain 51% of the network hash rate and possibly stage an attack.

## Technical implementation of ProgPoW

As ProgPoW follows the design of the original Ethash Proof-of-Work algorithm but tunes drastically by parameters and tunes applied for ASIC resistance, some core features are being inherited such as DAG cache verification and SHA3 hashes applied to the algorithm.

Following is the example code for DAG cache generation which is being changed every 30,000 blocks.

```c
void ethash_calculate_dag_item(
    node* const ret,
    uint32_t node_index,
    ethash_light_t const light
)
{
    uint32_t num_parent_nodes = (uint32_t) (light->cache_size / sizeof(node));
    node const* cache_nodes = (node const *) light->cache;
    node const* init = &cache_nodes[node_index % num_parent_nodes];
    memcpy(ret, init, sizeof(node));
    ret->words[0] ^= node_index;
    SHA3_512(ret->bytes, ret->bytes, sizeof(node));
    __m128i const fnv_prime = _mm_set1_epi32(FNV_PRIME);
    __m128i xmm0 = ret->xmm[0];
    __m128i xmm1 = ret->xmm[1];
    __m128i xmm2 = ret->xmm[2];
    __m128i xmm3 = ret->xmm[3];

    for (uint32_t i = 0; i != ETHASH_DATASET_PARENTS; ++i) {
        uint32_t parent_index = fnv_hash(node_index ^ i, ret->words[i % NODE_WORDS]) % num_parent_nodes;
        node const *parent = &cache_nodes[parent_index];

        {
            xmm0 = _mm_mullo_epi32(xmm0, fnv_prime);
            xmm1 = _mm_mullo_epi32(xmm1, fnv_prime);
            xmm2 = _mm_mullo_epi32(xmm2, fnv_prime);
            xmm3 = _mm_mullo_epi32(xmm3, fnv_prime);
            xmm0 = _mm_xor_si128(xmm0, parent->xmm[0]);
            xmm1 = _mm_xor_si128(xmm1, parent->xmm[1]);
            xmm2 = _mm_xor_si128(xmm2, parent->xmm[2]);
            xmm3 = _mm_xor_si128(xmm3, parent->xmm[3]);

            // have to write to ret as values are used to compute index
            ret->xmm[0] = xmm0;
            ret->xmm[1] = xmm1;
            ret->xmm[2] = xmm2;
            ret->xmm[3] = xmm3;
        }
    }
    SHA3_512(ret->bytes, ret->bytes, sizeof(node));
}
```

SHA3 can be used by following example

```c
def sha3(x):
    if isinstance(x, (int, long)):
        x = encode_int(x)
    return decode_int(utils.sha3(x))

def dbl_sha3(x):
    if isinstance(x, (int, long)):
        x = encode_int(x)
    return decode_int(utils.sha3(utils.sha3(x)))
```

ProgPoW has the following parameters for its design, The proposed settings have been tuned for a range of existing, commodity GPUs:

• PROGPOW\_PERIOD: Number of blocks before changing the random program • PROGPOW\_LANES: The number of parallel lanes that coordinate to calculate a single hash instance • PROGPOW\_REGS: The register file usage size • PROGPOW\_DAG\_LOADS: Number of uint32 loads from the DAG per lane • PROGPOW\_CACHE\_BYTES: The size of the cache • PROGPOW\_CNT\_DAG: The number of DAG accesses, defined as the outer loop of the algorithm \(64 is the same as ethash\) • PROGPOW\_CNT\_CACHE: The number of cache accesses per loop • PROGPOW\_CNT\_MATH: The number of math operations per loop

Following is the suggest parameter values for implementation.

| Parameter | ProgPoW |
| :--- | :--- |
| PROGPOW\_PERIOD | 50 |
| PROGPOW\_LANES | 16 |
| PROGPOW\_REGS | 32 |
| PROGPOW\_DAG\_LOADS | 4 |
| PROGPOW\_CNT\_DAG | 64 |
| PROGPOW\_CNT\_CACHE | 12 |
| PROGPOW\_CACHE\_BYTES | 16x1024 |
| PROGPOW\_CNT\_MATH | 20 |

To achieve more performance and to give more competitive advantage of performance per watt to AMD graphic cards compared with Nvidia graphic cards, following parameter values were proposed instead and it is likely to be used for unique value for EtherCore mainnet.

| Parameter | ProgPoW |
| :--- | :--- |
| PROGPOW\_PERIOD | 10 |
| PROGPOW\_LANES | 16 |
| PROGPOW\_REGS | 32 |
| PROGPOW\_DAG\_LOADS | 4 |
| PROGPOW\_CNT\_DAG | 64 |
| PROGPOW\_CNT\_CACHE | 11 |
| PROGPOW\_CACHE\_BYTES | 16x1024 |
| PROGPOW\_CNT\_MATH | 18 |

Some values were decreased to give more performance advantages for AMD cards, however, it doesn’t affect the ASIC resistance at all.

By reducing PROGPOW\_PERIOD, the hash rate of every period will naturally be slightly different \(+/- 5%\) due to the different random sequences generated. Reducing the period from ~10 minutes to ~2 minutes prevents the overall difficulty from drifting significantly in response to an individual period. As an added bonus this makes FPGAs even more impractical.

Typically compiling the kernel takes &lt;1 second, even on a relatively slow machine. In the case of a series of quick blocks and a system with an exceptionally slow CPU compiling for a few seconds every 10 blocks should not be a problem. This will especially be true once the planned Ethminer optimization that compiles period N+1’s kernel while period N is executing is in place.

By reducing the general computation requirements of ProgPoW, after testing on a wider variety of GPUs we’ve discovered the current parameters unintentionally causes some AMD GPUs to be compute limited instead of memory limited. Reducing the random cache and math counts by 10% increases the hash rate on those AMD GPUs. This has no effect on the hash rate of other AMD GPUs or any the Nvidia GPUs we tested.

This is the lowest the counts can be reduced while keeping the important property of \(PROGPOW\_DAG\_LOADS\(4\) + PROGPOW\_CNT\_CACHE\(11\) + PROGPOW\_CNT\_MATH\(18\) &gt; PROGPOW\_REGS\(32\)\). This property ensures that every mix\[\] destination gets written to.

The random program changes every PROGPOW\_PERIOD blocks to ensure the hardware executing the algorithm is fully programmable. If the program only changed every DAG epoch \(roughly 5 days\) certain miners could have time to develop hand-optimized versions of the random sequence, giving them an undue advantage.

All numerics are computed using unsigned 32 bit integers. Any overflows are trimmed off before proceeding to the next computation. Languages that use numerics not fixed to bit lengths \(such as Python and JavaScript\) or that only use signed integers \(such as Java\) will need to keep their languages’ quirks in mind. The extensive use of 32 bit data values aligns with modern GPUs internal data architectures.

ProgPoW algorithm uses a 32-bit variant of FNV1a for merging data. The one that is being used for Ethash algorithm which is FNV1 is in a deprecated state and it is the known part of weakness for ASIC development exploit, so FNV1a may provide better distribution properties.

```c
const uint32_t FNV_PRIME = 0x1000193;
const uint32_t FNV_OFFSET_BASIS = 0x811c9dc5;

uint32_t fnv1a(uint32_t h, uint32_t d)
{
    return (h ^ d) * FNV_PRIME;
}
```

ProgPow uses KISS99 for random number generation. This is the simplest \(fewest instruction\) random generator that passes the TestU01 statistical test suite. A more complex random number generator like Mersenne Twister can be efficiently implemented on a specialized ASIC, providing an opportunity for efficiency gains.

```c
typedef struct {
    uint32_t z, w, jsr, jcong;
} kiss99_t;

// KISS99 is simple, fast, and passes the TestU01 suite
// https://en.wikipedia.org/wiki/KISS_(algorithm)
// http://www.cse.yorku.ca/~oz/marsaglia-rng.html
uint32_t kiss99(kiss99_t &st)
{
    st.z = 36969 * (st.z & 65535) + (st.z >> 16);
    st.w = 18000 * (st.w & 65535) + (st.w >> 16);
    uint32_t MWC = ((st.z << 16) + st.w);
    st.jsr ^= (st.jsr << 17);
    st.jsr ^= (st.jsr >> 13);
    st.jsr ^= (st.jsr << 5);
    st.jcong = 69069 * st.jcong + 1234567;
    return ((MWC^st.jcong) + st.jsr);
}
```

The fill\_mix function populates an array of int32 values used by each lane in the hash calculations.

```c
void fill_mix(
    uint64_t hash_seed,
    uint32_t lane_id,
    uint32_t mix[PROGPOW_REGS]
)
{
    // Use FNV to expand the per-warp seed to per-lane
    // Use KISS to expand the per-lane seed to fill mix
    kiss99_t st;
    st.z = fnv1a(FNV_OFFSET_BASIS, seed);
    st.w = fnv1a(st.z, seed >> 32);
    st.jsr = fnv1a(st.w, lane_id);
    st.jcong = fnv1a(st.jsr, lane_id);
    for (int i = 0; i < PROGPOW_REGS; i++)
            mix[i] = kiss99(st);
}
```

Like Ethash SHA3 is used to seed the sequence per-nonce and to produce the final result. The Keccak-f800 variant is used as the 32-bit word size matches the native word size of modern GPUs. The implementation is a variant of SHAKE with width=800, bitrate=576, capacity=224, output=256, and no padding. The result of Keccak is treated as a 256-bit big-endian number - that is result byte 0 is the MSB of the value.

As with Ethash the input and output of the Keccak function are fixed and relatively small. This means only a single “absorb” and “squeeze” phase are required. For a pseudo-code implementation of the Keccak\_f800\_round function see the Round[b](https://github.com/ethercoreorg/docs/tree/79584a8fce8af087762514791ffbe1803d05f7f7/ethercore/A,RC/README.md) function in the “Pseudo-code description of the permutations” section of the official Keccak specs.

```c
hash32_t keccak_f800_progpow(hash32_t header, uint64_t seed, hash32_t digest)
{
    uint32_t st[25];

    // Initialization
    for (int i = 0; i < 25; i++)
        st[i] = 0;

    // Absorb phase for fixed 18 words of input
    for (int i = 0; i < 8; i++)
        st[i] = header.uint32s[i];
    st[8] = seed;
    st[9] = seed >> 32;
    for (int i = 0; i < 8; i++)
        st[10+i] = digest.uint32s[i];

    // keccak_f800 call for the single absorb pass
    for (int r = 0; r < 22; r++)
        keccak_f800_round(st, r);

    // Squeeze phase for fixed 8 words of output
    hash32_t ret;
    for (int i=0; i<8; i++)
        ret.uint32s[i] = st[i];

    return ret;
}
```

The inner loop uses FNV and KISS99 to generate a random sequence from the prog\_seed. This random sequence determines which mix state is accessed and what random math is performed.

Since the prog\_seed changes only once per PROGPOW\_PERIOD it is expected that while mining progPowLoop will be evaluated on the CPU to generate source code for that period’s sequence. The source code will be compiled on the CPU before running on the GPU.

```c
kiss99_t progPowInit(uint64_t prog_seed, int mix_seq_dst[PROGPOW_REGS], int mix_seq_src[PROGPOW_REGS])
{
    kiss99_t prog_rnd;
    prog_rnd.z = fnv1a(FNV_OFFSET_BASIS, prog_seed);
    prog_rnd.w = fnv1a(prog_rnd.z, prog_seed >> 32);
    prog_rnd.jsr = fnv1a(prog_rnd.w, prog_seed);
    prog_rnd.jcong = fnv1a(prog_rnd.jsr, prog_seed >> 32);
    // Create a random sequence of mix destinations for merge() and mix sources for cache reads
    // guarantees every destination merged once
    // guarantees no duplicate cache reads, which could be optimized away
    // Uses Fisher-Yates shuffle
    for (int i = 0; i < PROGPOW_REGS; i++)
    {
        mix_seq_dst[i] = i;
        mix_seq_src[i] = i;
    }
    for (int i = PROGPOW_REGS - 1; i > 0; i--)
    {
        int j;
        j = kiss99(prog_rnd) % (i + 1);
        swap(mix_seq_dst[i], mix_seq_dst[j]);
        j = kiss99(prog_rnd) % (i + 1);
        swap(mix_seq_src[i], mix_seq_src[j]);
    }
    return prog_rnd;
}
```

The math operations that merges values into the mix data are ones chosen to maintain entropy.

```c
// Merge new data from b into the value in a
// Assuming A has high entropy only do ops that retain entropy
// even if B is low entropy
// (IE don't do A&B)
uint32_t merge(uint32_t a, uint32_t b, uint32_t r)
{
    switch (r % 4)
    {
    case 0: return (a * 33) + b;
    case 1: return (a ^ b) * 33;
    // prevent rotate by 0 which is a NOP
    case 2: return ROTL32(a, ((r >> 16) % 31) + 1) ^ b;
    case 3: return ROTR32(a, ((r >> 16) % 31) + 1) ^ b;
    }
}
```

The math operations chosen for the random math are ones that are easy to implement in CUDA and OpenCL, the two main programming languages for commodity GPUs. The mul\_hi, min, clz, and popcount functions match the corresponding OpenCL functions. ROTL32 matches the OpenCL rotate function. ROTR32 is rotate right, which is equivalent to rotate\(i, 32-v\).

```c
// Random math between two input values
uint32_t math(uint32_t a, uint32_t b, uint32_t r)
{
    switch (r % 11)
    {
    case 0: return a + b;
    case 1: return a * b;
    case 2: return mul_hi(a, b);
    case 3: return min(a, b);
    case 4: return ROTL32(a, b);
    case 5: return ROTR32(a, b);
    case 6: return a & b;
    case 7: return a | b;
    case 8: return a ^ b;
    case 9: return clz(a) + clz(b);
    case 10: return popcount(a) + popcount(b);
    }
}
```

The flow of the inner loop is:

Lane \(loop % LANES\) is chosen as the leader for that loop iteration The leader’s mix\[0\] value modulo the number of 256-byte DAG entries is used to select where to read from the full DAG Each lane reads DAG\_LOADS sequential words, using \(lane ^ loop\) % LANES as the starting offset within the entry. The random sequence of math and cache accesses is performed The DAG data read at the start of the loop is merged at the end of the loop prog\_seed and loop come from the outer loop, corresponding to the current program seed \(which is block\_number/PROGPOW\_PERIOD\) and the loop iteration number. mix is the state array, initially filled by fill\_mix. dag is the bytes of the Ethash DAG grouped into 32 bit unsigned ints in litte-endian format. On little-endian architectures this is just a normal int32 pointer to the existing DAG.

DAG\_BYTES is set to the number of bytes in the current DAG, which is generated identically to the existing Ethash algorithm.

```c
void progPowLoop(
    const uint64_t prog_seed,
    const uint32_t loop,
    uint32_t mix[PROGPOW_LANES][PROGPOW_REGS],
    const uint32_t *dag)
{
    // dag_entry holds the 256 bytes of data loaded from the DAG
    uint32_t dag_entry[PROGPOW_LANES][PROGPOW_DAG_LOADS];
    // On each loop iteration rotate which lane is the source of the DAG address.
    // The source lane's mix[0] value is used to ensure the last loop's DAG data feeds into this loop's address.
    // dag_addr_base is which 256-byte entry within the DAG will be accessed
    uint32_t dag_addr_base = mix[loop%PROGPOW_LANES][0] %
        (DAG_BYTES / (PROGPOW_LANES*PROGPOW_DAG_LOADS*sizeof(uint32_t)));
    for (int l = 0; l < PROGPOW_LANES; l++)
    {
        // Lanes access DAG_LOADS sequential words from the dag entry
        // Shuffle which portion of the entry each lane accesses each iteration by XORing lane and loop.
        // This prevents multi-chip ASICs from each storing just a portion of the DAG
        size_t dag_addr_lane = dag_addr_base * PROGPOW_LANES + (l ^ loop) % PROGPOW_LANES;
        for (int i = 0; i < PROGPOW_DAG_LOADS; i++)
            dag_entry[l][i] = dag[dag_addr_lane * PROGPOW_DAG_LOADS + i];
    }

    // Initialize the program seed and sequences
    // When mining these are evaluated on the CPU and compiled away
    int mix_seq_dst[PROGPOW_REGS];
    int mix_seq_src[PROGPOW_REGS];
    int mix_seq_dst_cnt = 0;
    int mix_seq_src_cnt = 0;
    kiss99_t prog_rnd = progPowInit(prog_seed, mix_seq_dst, mix_seq_src);

    int max_i = max(PROGPOW_CNT_CACHE, PROGPOW_CNT_MATH);
    for (int i = 0; i < max_i; i++)
    {
        if (i < PROGPOW_CNT_CACHE)
        {
            // Cached memory access
            // lanes access random 32-bit locations within the first portion of the DAG
            int src = mix_seq_src[(mix_seq_src_cnt++)%PROGPOW_REGS];
            int dst = mix_seq_dst[(mix_seq_dst_cnt++)%PROGPOW_REGS];
            int sel = kiss99(prog_rnd);
            for (int l = 0; l < PROGPOW_LANES; l++)
            {
                uint32_t offset = mix[l][src] % (PROGPOW_CACHE_BYTES/sizeof(uint32_t));
                mix[l][dst] = merge(mix[l][dst], dag[offset], sel);
            }
        }
        if (i < PROGPOW_CNT_MATH)
        {
            // Random Math
            // Generate 2 unique sources
            int src_rnd = kiss99(prog_rnd) % (PROGPOW_REGS * (PROGPOW_REGS-1));
            int src1 = src_rnd % PROGPOW_REGS; // 0 <= src1 < PROGPOW_REGS
            int src2 = src_rnd / PROGPOW_REGS; // 0 <= src2 < PROGPOW_REGS - 1
            if (src2 >= src1) ++src2; // src2 is now any reg other than src1
            int sel1 = kiss99(prog_rnd);
            int dst  = mix_seq_dst[(mix_seq_dst_cnt++)%PROGPOW_REGS];
            int sel2 = kiss99(prog_rnd);
            for (int l = 0; l < PROGPOW_LANES; l++)
            {
                uint32_t data = math(mix[l][src1], mix[l][src2], sel1);
                mix[l][dst] = merge(mix[l][dst], data, sel2);
            }
        }
    }
    // Consume the global load data at the very end of the loop to allow full latency hiding
    // Always merge into mix[0] to feed the offset calculation
    for (int i = 0; i < PROGPOW_DAG_LOADS; i++)
    {
        int dst = (i==0) ? 0 : mix_seq_dst[(mix_seq_dst_cnt++)%PROGPOW_REGS];
        int sel = kiss99(prog_rnd);
        for (int l = 0; l < PROGPOW_LANES; l++)
            mix[l][dst] = merge(mix[l][dst], dag_entry[l][i], sel);
    }
}
```

The flow of the overall algorithm is:

* A Keccak hash of the header + nonce to create a seed
* Use the seed to generate initial mix data
* Loop multiple times, each time hashing random loads and random math into the mix data
* Hash all the mix data into a single 256-bit value
* A final Keccak hash is computed
* When mining this final value is compared against a hash32\_t target

```c
hash32_t progPowHash(
    const uint64_t prog_seed, // value is (block_number/PROGPOW_PERIOD)
    const uint64_t nonce,
    const hash32_t header,
    const uint32_t *dag // gigabyte DAG located in framebuffer - the first portion gets cached
)
{
    uint32_t mix[PROGPOW_LANES][PROGPOW_REGS];
    hash32_t digest;
    for (int i = 0; i < 8; i++)
        digest.uint32s[i] = 0;

    // keccak(header..nonce)
    hash32_t seed_256 = keccak_f800_progpow(header, nonce, digest);
    // endian swap so byte 0 of the hash is the MSB of the value
    uint64_t seed = bswap(seed_256[0]) << 32 | bswap(seed_256[1]);

    // initialize mix for all lanes
    for (int l = 0; l < PROGPOW_LANES; l++)
        fill_mix(seed, l, mix[l]);

    // execute the randomly generated inner loop
    for (int i = 0; i < PROGPOW_CNT_DAG; i++)
        progPowLoop(prog_seed, i, mix, dag);

    // Reduce mix data to a per-lane 32-bit digest
    uint32_t digest_lane[PROGPOW_LANES];
    for (int l = 0; l < PROGPOW_LANES; l++)
    {
        digest_lane[l] = FNV_OFFSET_BASIS
        for (int i = 0; i < PROGPOW_REGS; i++)
            digest_lane[l] = fnv1a(digest_lane[l], mix[l][i]);
    }
    // Reduce all lanes to a single 256-bit digest
    for (int i = 0; i < 8; i++)
        digest.uint32s[i] = FNV_OFFSET_BASIS;
    for (int l = 0; l < PROGPOW_LANES; l++)
        digest.uint32s[l%8] = fnv1a(digest.uint32s[l%8], digest_lane[l])

    // keccak(header .. keccak(header..nonce) .. digest);
    keccak_f800_progpow(header, seed, digest);
}
```

## Comparison with other Proof-of-Work algorithms

While ProgPoW is meant to be superior from gaining performance for commodity hardwares such as graphic cards, we have identified several PoW algorithms performance from developing mining specialized hardwares, these are the common assumptions made by the design of other algorithms.

SHA256

• Potential ASIC efficiency gain ~ 1000X

The SHA algorithm is a sequence of simple math operations - additions, logical ops, and rotates. To process a single op on a CPU or GPU requires fetching and decoding an instruction, reading data from a register file, executing the instruction, and then writing the result back to a register file. This takes significant time and power. A single op implemented in an ASIC takes a handful of transistors and wires. This means every individual op takes negligible power, area, or time. A hashing core is built by laying out the sequence of required ops. The hashing core can execute the required sequence of ops in much less time, and using less power or area, than doing the same sequence on a CPU or GPU. A Bitcoin ASIC consists of a number of identical hashing cores and some minimal off-chip communication.

Equihash

• Potential ASIC efficiency gain ~ 100X

The ~150mb of state is large but possible on an ASIC. The binning, sorting, and comparing of bit strings could be implemented on an ASIC at extremely high speed

CryptoNight

• Potential ASIC efficiency gain ~ 50X

Compared to Scrypt, CryptoNight does much less compute and requires a full 2mb of scratch pad \(there is no known Time/Memory Tradeoff attack\). The large scratch pad will dominate the ASIC implementation and limit the number of hashing cores, limiting the absolute performance of the ASIC. An ASIC will consist almost entirely of just on-die SRAM.

Ethash

• Potential ASIC efficiency gain ~ 2X

Ethash requires external memory due to the large size of the DAG. However that is all that it requires - there is minimal compute that is done on the result loaded from memory. As a result a custom ASIC could remove most of the complexity, and power, of a GPU and be just a memory interface connected to a small compute engine.

## Conclusion

ProgPoW utilizes almost all parts of a commodity GPU, excluding:

• The graphics pipeline \(displays, geometry engines, texturing, etc\); • Floating point math.

Making use of either of these would have significant portability issues between commodity hardware vendors, and across programming languages. Since the GPU is almost fully utilized, there’s little opportunity for specialized ASICs to gain efficiency. Removing both the graphics pipeline and floating point math could provide up to 1.2x gains in efficiency, compared to the 2x gains possible in Ethash, and 50x gains possible for CryptoNight.

