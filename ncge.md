---
layout: default
---

# Non-commutative gaussian elimination (from the course "groups, rings and fields" - taught by Prof. Dror Bar-Natan)

<div id="rubiks-net-wrapper" style="font-family: sans-serif; max-width: 600px; margin: 30px auto; padding: 20px; border: 1px solid #ddd; border-radius: 8px; background: #fff; box-shadow: 0 4px 6px rgba(0,0,0,0.05);">
  <h3 style="margin-top: 0; text-align: center; border-bottom: 1px solid #eee; padding-bottom: 10px;">NCGE Rubik's Cube Numbering (1-54)</h3>
  <p style="text-align: center; color: #666; font-size: 14px; margin-bottom: 25px; line-height: 1.5;">
    This 2D net shows how the 54 stickers are mapped to natural numbers. <br>
    <em>Notice how the Front (White) face contains stickers 13-15, 22-24, and 31-33.</em>
  </p>
  
  <div id="rubiks-grid" style="display: grid; grid-template-columns: repeat(3, 90px); grid-template-rows: repeat(4, 110px); gap: 10px; justify-content: center; margin: 0 auto;">
    </div>

  <script>
    (function() {
      const container = document.getElementById('rubiks-grid');
      
      // The exact layout mapped from the permutation cycles
      const faces = [
        { name: 'Up (U)', col: 2, row: 1, bg: '#0046ad', color: '#fff', nums: [1,2,3,4,5,6,7,8,9] },
        { name: 'Left (L)', col: 1, row: 2, bg: '#ff8a00', color: '#000', nums: [10,11,12,19,20,21,28,29,30] },
        { name: 'Front (F)', col: 2, row: 2, bg: '#ffffff', color: '#000', nums: [13,14,15,22,23,24,31,32,33] },
        { name: 'Right (R)', col: 3, row: 2, bg: '#b71234', color: '#fff', nums: [16,17,18,25,26,27,34,35,36] },
        { name: 'Down (D)', col: 2, row: 3, bg: '#009b48', color: '#fff', nums: [37,38,39,40,41,42,43,44,45] },
        { name: 'Back (B)', col: 2, row: 4, bg: '#ffd500', color: '#000', nums: [46,47,48,49,50,51,52,53,54] } 
      ];

      faces.forEach(face => {
        // Wrapper to hold both the label and the 3x3 face
        const wrapper = document.createElement('div');
        wrapper.style.gridColumn = face.col;
        wrapper.style.gridRow = face.row;
        wrapper.style.display = 'flex';
        wrapper.style.flexDirection = 'column';
        wrapper.style.alignItems = 'center';

        // Face Label
        const label = document.createElement('div');
        label.innerText = face.name;
        label.style.fontSize = '13px';
        label.style.fontWeight = 'bold';
        label.style.color = '#555';
        label.style.marginBottom = '6px';
        wrapper.appendChild(label);

        // The 3x3 Grid for the stickers
        const faceDiv = document.createElement('div');
        faceDiv.style.display = 'grid';
        faceDiv.style.gridTemplateColumns = 'repeat(3, 1fr)';
        faceDiv.style.gridTemplateRows = 'repeat(3, 1fr)';
        faceDiv.style.gap = '2px';
        faceDiv.style.background = '#222'; // Acts as the black plastic border
        faceDiv.style.border = '3px solid #222';
        faceDiv.style.borderRadius = '4px';
        faceDiv.style.width = '100%';
        faceDiv.style.aspectRatio = '1 / 1';
        faceDiv.style.boxSizing = 'border-box';

        // Populate the 9 stickers
        face.nums.forEach(num => {
          const sticker = document.createElement('div');
          sticker.style.background = face.bg;
          sticker.style.color = face.color;
          sticker.style.display = 'flex';
          sticker.style.justifyContent = 'center';
          sticker.style.alignItems = 'center';
          sticker.style.fontSize = '14px';
          sticker.style.fontWeight = 'bold';
          sticker.style.userSelect = 'none';
          sticker.innerText = num;
          faceDiv.appendChild(sticker);
        });

        wrapper.appendChild(faceDiv);
        container.appendChild(wrapper);
      });
    })();
  </script>
</div>

## Context

Non-commutative gaussian elimination (which I will refer to as NCGE from here on out) was taught to us in the beginning of "groups, rings and fields" (aka MAT347). We were introduced to the problem like so: consider a standard 3 x 3 x 3 Rubik's Cube. Label each sticker on the cube by the natural numbers from 1 to 54. Now, consider a group $G = \langle g_1, g_2, g_3, g_4, g_5, g_6 \rangle$, where each of the six generators constitutes one of the six 90 degree turns that are on unique axes. As we can express each $g_i$ as a cycle of natural numbers (for example, turning the white face 90 degrees clockwise would be expressed as (13, 15, 33, 31)(14, 24, 32, 22)(7, 16, 39, 30)(8, 25, 38, 21)(9, 34, 37, 12)), each $g_i$ is a member of the permutation group $S_54$.

Having defined a subgroup of $S_{54}$ with an unknown size, an obvious question is: if we pick a random element of $S_{54}$, how can we find out whether it is an element of $G$ or not? In other words, which permutations of the 54 stickers are valid Rubik's Cube combinations that can be achieved by using the six unique-axis turns, and which can only be achieved by peeling off the stickers and placing them where we want them to go?

## In comes NCGE (how the method works)

Prepare a lower-half-left triangular table with 54 columns and 54 rows. Naturally, the left most column will be the tallest (54 boxes), while the right most column has just one box. Label each box $(i, j)$, with $i$ being the column number, and $j$ being the row number. Based on how we define the table, $i \leq j$.

Start by feeding in each of the generator elements into the table. This "feeding" process for non-identity elements works as follows:

1. find the pivotal position $i$ in that generator element $g_k$ (this is the first number from 1 to 54 that is mapped to a number that is not itself in the permutation; the reason we know it must be mapped to a number ahead of it is: if $i > j$, then $g_k(i) = j$ and $g_k(j)=j$, contradicting it being a permutation (bijection))
2. if box $(i, j)$ is empty, place $g_k$ in that box. If not, go to step 3.
3. go back to step 1 and feed $g' = \sigma g_k$, where $\sigma$ is the element already in $(i, j)$ (you do this over and over until you either place the new permutation in a box of reach the identity).
4. professor Bar-Natan called this "the twist": Having done all the generators, for every occupied $(i, j)$ and $(r, s)$, go back to step 1 and feed in $\sigma_{i, j} \sigma_{r, s}$ (these of course refer to the permutation stored in those boxes after step 3). Repeat this process until the table stops changing.

In class, we covered several interesting claims about this process and the resulting table. I want to focus on the claim I found the most interesting:

Claim: Feed any element of $S_{54}$ into the table. If it eventually becomes the identity permutation, it is a valid Rubik's Cube permutation. If it reaches an empty box, it is not a valid Rubik's Cube permutation.

## Proof

### 1. If feeding ends at the identity, then $\sigma \in G$.

The algorithm produces a sequence  
$$
\sigma = \tau_1 \tau_2 \cdots \tau_k \cdot (\text{identity})
$$
where each $\tau_m$ is the element taken from a box $(i_m, j_m)$ during the reduction.  
By construction, every box in $T$ is obtained from the generators $g_1,\dots,g_6$ using only multiplication and inversion (the “feeding” and “twist” steps). Hence every $\tau_m \in G$.  
Therefore $\sigma = \tau_1 \tau_2 \cdots \tau_k \in G$.

### 2. If feeding reaches an empty box, then $\sigma \notin G$.

Suppose for contradiction that $\sigma \in G$.  
Because the table $T$ is closed under the operations that built it (proved during lecture - I will add a short proof of this later), every element of $G$ can be written as a monotone product.  
$$
\sigma = \sigma_{1,j_1}\, \sigma_{2,j_2} \cdots \sigma_{54,j_{54}}
$$ 
where each $\sigma_{i,j_i}$ belongs to box $(i,j_i)$ of $T$ (or is the identity if the box is empty).  
Now feed this product into the table. The algorithm will read the factors from left to right, reducing the pivot at each step. If some box $(i,j_i)$ were empty, the reduction would stop before consuming all factors, contradicting the assumption that $\sigma$ reduces completely to the identity (since $\sigma \in G$ must reduce to identity by the first part of the claim). Hence all needed boxes must be non‑empty. Therefore, if the feeding ever encounters an empty box, $\sigma$ cannot be in $G$.

Thus, the feeding test is both necessary and sufficient for membership in the Rubik’s cube group $G$.

## Mini-example with $S_4$ instead of $S_{54}$

<div id="ncge-container" style="font-family: sans-serif; border: 1px solid #ddd; padding: 20px; border-radius: 8px; background: #fff; color: #333; max-width: 800px; margin: 20px auto;">
  <h3 style="margin-top: 0; border-bottom: 1px solid #eee; padding-bottom: 10px;">NCGE for S<sub>4</sub> - Step-by-Step</h3>
  
  <div id="ncge-table" style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; margin-bottom: 20px;">
    </div>
  
  <div style="min-height: 80px; padding: 15px; background: #f9f9f9; border-left: 4px solid #007bff; margin-bottom: 15px; border-radius: 0 4px 4px 0;">
    <p id="ncge-explanation" style="margin: 0; font-size: 15px; line-height: 1.5;"></p>
  </div>
  
  <div style="display: flex; gap: 10px;">
    <button id="btn-back" style="padding: 8px 16px; cursor: pointer; border: 1px solid #ccc; background: #fff; border-radius: 4px;" disabled>Back</button>
    <button id="btn-next" style="padding: 8px 16px; cursor: pointer; border: none; background: #007bff; color: white; border-radius: 4px;">Next</button>
  </div>

  <script>
    (function() {
      const container = document.getElementById('ncge-table');
      
      // Build the lower triangular matrix
      for (let j = 1; j <= 4; j++) {
        for (let i = 1; i <= 4; i++) {
          const cell = document.createElement('div');
          cell.id = `box-${i}-${j}`;
          cell.style.border = '1px solid #ccc';
          cell.style.padding = '10px';
          cell.style.textAlign = 'center';
          cell.style.minHeight = '45px';
          cell.style.borderRadius = '4px';
          cell.style.transition = 'background-color 0.3s';
          
          if (i < j) {
            // Valid boxes to be filled with permutations
            cell.innerHTML = `<small style="color:#888; font-size: 12px;">(${i},${j})</small><br><span class="content" style="font-family: monospace; font-size: 14px; font-weight: bold;"></span>`;
            cell.style.backgroundColor = '#fafafa';
          } else if (i === j) {
            // The diagonal: Identity
            cell.innerHTML = `<small style="color:#888; font-size: 12px;">(${i},${j})</small><br><span class="content" style="font-family: monospace; font-size: 14px; font-weight: bold; color: #888;">I</span>`;
            cell.style.backgroundColor = '#efefef';
          } else {
            // Upper triangle is hidden
            cell.style.visibility = 'hidden'; 
          }
          container.appendChild(cell);
        }
      }

      // Mathematically verified steps with Right-to-Left composition
      const steps = [
        {
          text: "<strong>Initial State:</strong> Based on the notes, we prepare a lower-triangular table. The diagonal (i=j) represents the Identity 'I'. We feed in generators g<sub>1</sub> = (1,2,3,4) and g<sub>2</sub> = (1,2).",
          updates: {}, highlight: null
        },
        {
          text: "<strong>Step 1 (Feeding g<sub>1</sub>):</strong> Feed g<sub>1</sub> = (1,2,3,4). The first element moved is 1, mapping to 2. Pivot is i=1, j=2.",
          updates: {}, highlight: "box-1-2"
        },
        {
          text: "<strong>Step 2:</strong> Box (1,2) is empty. We place (1,2,3,4) inside.",
          updates: {"box-1-2": "(1,2,3,4)"}, highlight: "box-1-2"
        },
        {
          text: "<strong>Step 3 (Feeding g<sub>2</sub>):</strong> Feed g<sub>2</sub> = (1,2). The pivot is i=1, mapping to j=2. However, box (1,2) is occupied by σ = (1,2,3,4).",
          updates: {}, highlight: "box-1-2"
        },
        {
          text: "<strong>Step 4:</strong> We compute σ<sup>-1</sup>g<sub>2</sub> = (1,4,3,2)(1,2) = (2,4,3). We now feed (2,4,3). The first element moved is 2, mapping to 4. Pivot i=2, j=4.",
          updates: {}, highlight: "box-2-4"
        },
        {
          text: "<strong>Step 5:</strong> Box (2,4) is empty. We place (2,4,3) inside.",
          updates: {"box-2-4": "(2,4,3)"}, highlight: "box-2-4"
        },
        {
          text: "<strong>Step 6 (The Twist):</strong> Now we feed combinations of occupied boxes. We feed σ<sub>2,4</sub>σ<sub>1,2</sub> = (2,4,3)(1,2,3,4) = (1,4). The pivot is i=1, mapping to j=4.",
          updates: {}, highlight: "box-1-4"
        },
        {
          text: "<strong>Step 7:</strong> Box (1,4) is empty. We place (1,4) inside.",
          updates: {"box-1-4": "(1,4)"}, highlight: "box-1-4"
        },
        {
          text: "<strong>Step 8 (The Twist):</strong> Next, we feed σ<sub>2,4</sub>σ<sub>1,4</sub> = (2,4,3)(1,4) = (1,3,2,4). The pivot is i=1, mapping to j=3.",
          updates: {}, highlight: "box-1-3"
        },
        {
          text: "<strong>Step 9:</strong> Box (1,3) is empty. We place (1,3,2,4) inside.",
          updates: {"box-1-3": "(1,3,2,4)"}, highlight: "box-1-3"
        },
        {
          text: "<strong>Step 10 (Collision!):</strong> We feed σ<sub>1,2</sub>σ<sub>1,3</sub> = (1,2,3,4)(1,3,2,4) = (1,4,2). Pivot i=1 maps to j=4. We look at box (1,4) but it is already occupied by (1,4).",
          updates: {}, highlight: "box-1-4"
        },
        {
          text: "<strong>Step 11:</strong> We compute (1,4)<sup>-1</sup>(1,4,2) = (2,4). Feed (2,4). Pivot i=2 maps to j=4. But box (2,4) is already occupied by (2,4,3).",
          updates: {}, highlight: "box-2-4"
        },
        {
          text: "<strong>Step 12:</strong> We compute (2,4,3)<sup>-1</sup>(2,4) = (3,4). Feed (3,4). Pivot i=3 maps to j=4. Box (3,4) is empty!",
          updates: {}, highlight: "box-3-4"
        },
        {
          text: "<strong>Step 13:</strong> We place (3,4) into box (3,4).",
          updates: {"box-3-4": "(3,4)"}, highlight: "box-3-4"
        },
        {
          text: "<strong>Step 14 (The Final Box):</strong> We feed σ<sub>3,4</sub>σ<sub>2,4</sub> = (3,4)(2,4,3) = (2,3). Pivot i=2 maps to j=3. Box (2,3) is empty.",
          updates: {}, highlight: "box-2-3"
        },
        {
          text: "<strong>Step 15:</strong> We place (2,3) into box (2,3).",
          updates: {"box-2-3": "(2,3)"}, highlight: "box-2-3"
        },
        {
          text: "<strong>NCGE Complete:</strong> All 6 lower-triangular boxes are filled. Any further twist combinations fed into this table will map back onto themselves completely, dropping straight through to the Identity (I) diagonal.",
          updates: {}, highlight: null
        }
      ];

      let currentStep = 0;

      function renderStep(stepIndex) {
        // Clear active text but leave "I" on diagonals
        for(let j=1; j<=4; j++) {
          for(let i=1; i<=4; i++) {
            if (i < j) {
              let el = document.getElementById(`box-${i}-${j}`);
              if(el) {
                  el.querySelector('.content').innerText = '';
                  el.style.backgroundColor = '#fafafa';
              }
            } else if (i === j) {
              let el = document.getElementById(`box-${i}-${j}`);
              if(el) el.style.backgroundColor = '#efefef';
            }
          }
        }
        
        // Rebuild state
        for (let k = 0; k <= stepIndex; k++) {
          let s = steps[k];
          for (let box in s.updates) {
             document.getElementById(box).querySelector('.content').innerText = s.updates[box];
          }
        }
        
        // Apply highlights
        let cur = steps[stepIndex];
        document.getElementById('ncge-explanation').innerHTML = cur.text;
        if (cur.highlight) {
           document.getElementById(cur.highlight).style.backgroundColor = '#fff3cd'; // Highlight color
        }

        document.getElementById('btn-back').disabled = (stepIndex === 0);
        document.getElementById('btn-next').disabled = (stepIndex === steps.length - 1);
        document.getElementById('btn-back').style.opacity = (stepIndex === 0) ? '0.5' : '1';
        document.getElementById('btn-next').style.opacity = (stepIndex === steps.length - 1) ? '0.5' : '1';
      }

      document.getElementById('btn-next').addEventListener('click', () => {
        if (currentStep < steps.length - 1) {
          currentStep++;
          renderStep(currentStep);
        }
      });

      document.getElementById('btn-back').addEventListener('click', () => {
        if (currentStep > 0) {
          currentStep--;
          renderStep(currentStep);
        }
      });

      renderStep(0);
    })();
  </script>
</div>

## Taking it further (my personal additions/interests)

### The order of the group $G$

I will start with something fairly simple, that being finding how many elements the group $G = \langle g_1, g_2, g_3, g_4, g_5, g_6 \rangle$ has. Although I did not state this as an objective in the initial sections, I touched upon it being an unknown. Conveniently, the NCGE process gives us a simple way of finding the order of $G$. If we add up the number of filled boxes in each column, and multiply the totals for each column together, we get $\|G\|$.

Well, why is this the case? The answer lies in a concept that could be called the "stabilizer chain", although it really relies on the more commonly known "orbit-stabilizer theorem".

Let $G_1 = G$ be the entire group. Now, define $G_2$ as the subgroup of $G_1$ containing all permutations that fix the number 1 in its place (in other words, $\sigma(1) = 1$). Next, define $G_3$ as the subgroup of $G_2$ that also fixes 2 (a quick justification for these being subgroups - they are trivially subsets, and satisfy the subgroup tests since composition and inverse will both fix the same elements).

Continuing this process, let $G_i$ be the subgroup of elements in $G$ that fix all numbers from 1 to $i - 1$. This creates a nested chain of subgroups until we eventually reach the identity element (or trivial subgroup that fixes every number from 1 to 54):

$$
G = G_1 \geq G_2 \geq G_3 \geq \dots \geq G_{54} = \{e\}
$$

Having established this, we can now use the orbit-stabilizer theorem. For the purpose of recall (partially so I get this right as well haha), if a group $H$ acts on a set, then for any element $x$ in that set:

$$
|H| = |\text{Orbit}_H(x)| \times |\text{Stabilizer}_H(x)|
$$

Applying this to the subgroup $G_i$ acting on the number $i$:

- The stabilizer of $i$ in $G_i$ is the set of elements that fix $1, \dots, i - 1$ and also fix $i$. This is the same as the next subgroup in the chain defined above: $G_{i + 1}$.
- The orbit of $i$ under $G_i$, which we can call $\text{Orb}_{G_i}(i)$, is the set of all possible destinations $j$ that $i$ can be sent to by elements of $G_i$.

Substituting this into the orbit-stabilize equation gives us the following recursive formula:

$$
|G_i| = |\text{Orb}_{G_i}(i)| \times |G_{i + 1}|
$$

Now, we connect this to the NCGE table. To do this, we must ask the question: what is column $i$ in the table? By the NCGE rules established earlier, any permutation placed in column $i$ has $i$ as its pivot position, meaning it fixes every number strictly less than $i$. Therefore, every element in column $i$ belongs to $G_i$. In addition to this, we may say that a box $(i, j)$ is filled iff there is some permutation in $G_i$ that maps $i$ to $j$. Hence, the number of occupied boxes in column $i$ is exactly the size of the orbit $\|\text{Orb}_{G_i}(i)\|$.

Using all of this, we may do the final computation as follows:

$$
\begin{flalign*}
|G_1| &= |\text{Orb}_{G_1}(1)| \times |G_2|&&\\
|G_1| &= |\text{Orb}_{G_1}(1)| \times |\text{Orb}_{G_2}(2)| \times |G_3|&&\\
\dots&&\\
|G_1| &= |\text{Orb}_{G_1}(1)| \times |\text{Orb}_{G_2}(2)| \times \dots \times |\text{Orb}_{G_{54}}(54)| \times |\{e\}|&&\\
\end{flalign*}
$$

Since the size of each orbit is the number of filled boxes in that column, $\|\{e\}\|$ is 1, and $G = G_1$, we get that:

$$
|G| = \prod_{i=1}^{54} \text{Number of occupied boxes in column i}
$$

and we are done.

[back](./)