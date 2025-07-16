These are the legacy scripts that I wrote while preparing my paper "The syzygy distinguisher" in order to experiment and compute examples.

Computer algebra systems such as Macaulay2, Magma, or Singular do provide commands to compute Betti numbers or minimal resolutions, but these are somehow too general and they crash even when applied to codes of very moderate parameter size.  
This forced me to write these dedicated scripts. For some reason, I wrote them in Magma.  
These scripts were first written only for myself, and were not planned to be shared. This explains their lack of readability, for which I should now apologize.  
Also, the algorithms implemented are very basic. My only aim was to quickly get something that just works. Many improvements and optimizations can be devised, and actually such a project is currently under development (starting with N.Marteau and P.Perrier's internship).

The main command is the 'Betti' command in the "resout.magma" file:
```
Betti(G,r);
```
computes the first row of the Betti diagram of the linear code with generator matrix G, up to degree r.  
It then completes the second row of the Betti diagram, up to degree r, pretending that the code has regularity 2. If this is not true, then this second row is just crap and should be ignored (a warning " !! k2 < n" appears).

Example:
```
> load "resout.magma"; 
Loading "resout.magma"
Loading "produits.magma"
Loading "macaulaymatrix.magma"
> C:=GolayCode(GF(2),false);
> G:=GeneratorMatrix(C);
> k:=Dimension(C);
> Betti(G,k);
indices = [ 55, 319, 880, 1353, 990, -330, -1617, -1870, -1221, -485, -110, -11 ]
b_{1,2} = 55
b_{2,3} = 320
b_{3,4} = 891
b_{4,5} = 1408
b_{5,6} = 1210
b_{6,7} = 320
b_{7,8} = 55
b_{8,9} = 0
b_{9,10} = 0
b_{10,11} = 0
b_{11,12} = 0
[   1    0    0    0    0    0    0    0    0    0    0    0]
[   0   55  320  891 1408 1210  320   55    0    0    0    0]
[   0    1   11   55  220  650 1672 1870 1221  485  110   11]
```

For more cryptographically motivated examples, first choose parameters such as:
```
m:=8; t:=6; s:=18; r:=4;
```
then either generate a s-shortened binary Goppa code with these parameters:
```
load "codesalgebriques.magma";
F:=GF(2); n:=2^m;
K<b>:=ext<F|m>;
S:=Setseq(RandomSubset(Set(K),n));
repeat
p:=Random(Sym(n));
S:=[S[i^p]: i in [1..n]];
R<z>:=PolynomialRing(K);
repeat
g:=z^t;
for i in [0..t-1] do
g:=g+Random(K)*z^i;
end for;
until IsIrreducible(g);
G:=MatricePariteStandardGoppa(S,g: F:=F);
G:=ReduitMatrice(G,F);
G:=SubmatrixRange(G,s+1,s+1,m*t,n);
until (Rank(G) eq m*t-s) and (Rank(CarreMatrice(G)) eq n-s);
```
or generate a random systematic matrix of the same dimensions:
```
F:=GF(2); n:=2^m;
repeat
G:=HorizontalJoin(IdentityMatrix(F,m*t-s),RandomMatrix(F,m*t-s,n-m*t));
until (Rank(G) eq m*t-s) and (Rank(CarreMatrice(G)) eq n-s); 
```
then run:
```
Betti(G,r: memory:=true,timing:=true,stats:=true,fichier:="outfile.txt");
```
and observe in the outfile.txt that the b_{r-1,r} is zero in the random case, and nonzero in the Goppa case.

There is also a command 'Beta3' in the "beta3.magma" file that computes the degree 3 Betti number b_{1,3} (which determines b_{2,3} since the difference between them only depends on the parameters).  
It allows to play with the ordering of the rows and the orderings of the basis of degree 2 and 3 monomials in the Macaulay matrix, and observe what happens when the computation proceeds. It turns out that this is how I first discovered that shortening was so helpful for the distinguishing problem.     
For some accidental reason, 'Beta3' works with the dual code, while 'Betti' works with the primal (sorry for that).  

Example:
```
load "beta3.magma";
C:=BKLC(GF(2),45,25);
Beta3(C: ordre2:="lex+", ordre3:="lex-", tri:="tete-");
Beta3(C: ordre1:="+", ordre2:="rlx-", tri:="naif21+");
```
