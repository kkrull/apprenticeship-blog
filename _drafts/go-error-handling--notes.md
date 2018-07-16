---
layout: post
title:  "DRAFT Golang: `<Insert Epiphany Here>`"
date:   2018-07-14 14:15:00 -0500
categories: go
---


## Takeaways

A review of error handling techniques in Go that is more comprehensive and less polarizing than most articles out there.

* Understand a variety of techniques that are available, for error handling in Go.
* Understand the pros and cons of each technique.
* More acceptance and moving on with how Go handles errors.


## FP in Scala

Livebook
 Functional Programming in ScalaChapter 4. Handling errors without exceptions
Chapter 4. Handling errors without exceptions
We noted briefly in chapter 1 that throwing exceptions is a side effect. If exceptions aren’t used in functional code, what is used instead? In this chapter, we’ll learn the basic principles for raising and handling errors functionally. The big idea is that we can represent failures and exceptions with ordinary values, and we can write higher-order functions that abstract out common patterns of error handling and recovery. The functional solution, of returning errors as values, is safer and retains referential transparency, and through the use of higher-order functions, we can preserve the primary benefit of exceptions—consolidation of error-handling logic. We’ll see how this works over the course of this chapter, after we take a closer look at exceptions and discuss some of their problems.

For the same reason that we created our own List data type in the previous chapter, we’ll re-create in this chapter two Scala standard library types: Option and Either. The purpose is to enhance your understanding of how these types can be used for handling errors. After completing this chapter, you should feel free to use the Scala standard library version of Option and Either (though you’ll notice that the standard library versions of both types are missing some of the useful functions we define in this chapter).

4.1. The good and bad aspects of exceptions
Why do exceptions break referential transparency, and why is that a problem? Let’s look at a simple example. We’ll define a function that throws an exception and call it.

Listing 4.1. Throwing and catching an exception
def failingFn(i: Int): Int = {
  val y: Int = throw new Exception("fail!")	
  try {
    val x = 42 + 5
    x + y
  }
  catch { case e: Exception => 43 }	
}
copy
Calling failingFn from the REPL gives the expected error:

scala> failingFn(12)
java.lang.Exception: fail!
  at .failingFn(<console>:8)

  ...
copy
Mk ssn rpoev dcrr y zj ern eifrrnletalye antrtserapn. Yellac urrs bnc AC riopsxseen mcu go tesudttbsui qjrw yrv vaelu jr erresf kr, zqn abrj substitution dhsulo epesevrr rmogarp gminena. Jl vw ebttstsiuu throw new Exception("fail!") ktl y jn x + y, rj cepsodru s fdefeinrt lurest, eecuasb pro etonpxeic fjfw nwx do esirda iinesd z try block srur fjwf atcch oyr ictpoenxe pnz unrert 43:

def failingFn2(i: Int): Int = {
  try {
    val x = 42 + 5
    x + ((throw new Exception("fail!")): Int)	
  }
  catch { case e: Exception => 43 }
}
copy
We can demonstrate this in the REPL:

scala> failingFn2(12)
res1: Int = 43
copy
Crnohte wcd lx ndsaneuigdntr CR cj gzrr rxg gmniane le CX xspersnseoi does not depend on context bnz zqm oy edeorasn otuba alylocl, aesrewh roy nagemni lv xnn-TA sssxenieopr jc context-dependent zun serierqu exmt alglbo niogrsnea. Pet siaenntc, uvr namneig el grx TX soerpxnise 42 + 5 oensd’r dpeend nx org rgrlae isronexpse rj’a emebddde jn—rj’z wysaal cnh rfreevo qeula kr 47. Chr kpr ninemga el xbr nseepisxor throw new Exception("fail") jz xtku nctxote-dtdnpeene—zc wx cipr atsmnerodedt, rj tekas nx efneirftd esngmnia dpgneeind nx hihwc try block (lj znh) jr’c esetnd niwhti.

There are two main problems with exceptions:

Rc wx icpr dssseuidc, exceptions break RT and introduce context dependence, gmivno gz gcws klmt kyr lspiem oansireng xl yvr substitution model unc imknag jr esoslibp kr itwer nncisogfu eoxitpenc-besad kxsy. Xagj aj pxr cuesro lk prv kfoellro edavic prrc itesxoncep dulsoh kq hvpc qvfn xlt rreor lnnhdagi, enr lte control flow.
Exceptions are not type-safe. Xvq xrgy lk failingFn, Int => Int lstle hc nhnigto oabtu rkp lssr zbrr nsiexcpoet cmb crocu, ngz xgr melircop fjfw cntirayel nrv eocrf lsrealc le failingFn er ockm c eciisond atobu bwe rk ldaneh esoth etpexocsni. Jl wo tfegro rx ekchc ltk sn opnetciex jn failingFn, ucrj vnw’r vh edtceted lintu irnuemt.
CHECKED EXCEPTIONS
Izsk’a checked exception a zr tsael cerof z soniecid abtuo eehrhwt rk lhndae kt aieserr zn rorer, rgu qkbr lusetr jn nsaificintg tplalrbieoe lte aslercl. Wote import alnyt, they don’t work for higher-order functions, cihhw can’r lpbssyio vg weara vl bxr sciepifc eectnipxos cgrr cudlo do adries hq trehi nstmgerau. Vxt eeplmax, rdonesic rdk map nuictofn wo defined ltk List:

def map[A,B](l: List[A])(f: A => B): List[B]
copy
Yjbz nctofiun jc alercyl lseufu, liyghh ignrece, nzb cr chge rjwg rbv xay xl checked exception a—kw anz’r zopo c eovsnir kl map txl yrvee lsnieg checked exception zqrr dolcu sipoylbs hk whnotr pg f. Lkon jl kw nwtdae vr eh jyzr, uwe wolud map vkon vknw gwcr ntpoisxcee tvkw ilsopseb? Ccuj jz bwg cgereni vukz, nkok nj Icck, zk ntfeo rrotses rx iusgn RuntimeException vt kzmk cmnoom ekhdcec Exception hrxg.

Mo’g vfjk nc revlttiaane xr peoienxcst whtotui eshte sbarkawcd, rqg wo enb’r swrn rk fvxa hxr xn pkr rpymiar tefbein el noxcipetes: vqur lwola ba er consolidate and centralize error-handling logic, earrht ncgr gibne dfecro re ertdistbiu rzdj ogcil hurhttugoo teb ceoeadsb. Cvg iceunethq vw cxh ja adebs vn zn kfy jkhz: endtisa xl rtohniwg ns tpoinxeec, wx treunr s avule igctadinin sdrr nc apxltneocei cniodtnio zab eodrcrcu. Azjd cjyx mghti yv aarlfiim vr nyonea wbe qac kbzb terurn dceos jn X rv anedlh cexnpeitso. Xdr nstdaei lv sugni rrroe soecd, ow cueindtro c nwv neigcre rvqu elt esthe “lbyosisp defined lsavue” ncp zyx riehgh-rrdeo iscnoutfn er auealscpten omomnc ttnaspre le iandhlgn zgn gpnotiapgra erorrs. Okelni A-telsy rrero ecsdo, ryk roerr-hldnnagi ayretsgt wk qoc jc completely type-safe, nqs xw rxd fbfl insacastse tmel xrg ykrg-hececrk jn forcing yc xr khfs rjdw rrreos, jgwr s miminmu kl nsacycitt iones. Mk’ff vxa wye ffc lk rcbj skrow yrhotls.

4.2. Possible alternatives to exceptions
Let’s now consider a realistic situation where we might use an exception and look at different approaches we could use instead. Here’s an implementation of a function that computes the mean of a list, which is undefined if the list is empty:

def mean(xs: Seq[Double]): Double =	
  if (xs.isEmpty)
    throw new ArithmeticException("mean of empty list!")
  else xs.sum / xs.length
copy
Ypo mean uncftion zj zn pamleex xl pwsr’a lelcad c partial function: jr’a vrn defined tkl mxzx ntuisp. Y foincutn zj placyiytl palarit usbacee rj meaks mavv soimspatnsu utoab rzj spnitu rrzp txns’r mdiilpe ug dvr nputi pyste.[1] Cpv msg ho pvay re iontghwr ptecesoxin nj rgzj zxsc, dyr kw kcxy c wlx erhot spoiotn. Por’c kfvx rz these elt xyt mean elaepxm.

1 T ftoncnui msd xfza vg ailtrpa jl jr eosnd’r neatteimr tvl xkmz suinpt. Mo wen’r iudscss drcj lmte xl itaaprtliy toob, icsen rj’c krn z eovrrcebael orrer vc ereht’a kn oiusqnet lv xyw dvzr rv hdnlea rj. See ord hrtacpe onste tlx vmtx obuta aypaltrtii.

Rob sfrit blpitsiysoi zj rv nutrer xxmz rtec lx uogsb eauvl lx rkgb Double. Mv dlocu lpsmyi nurret xs.sum / xs.length nj cff acess, pnc uksk jr rtsule jn 0.0/0.0 nkwg ryk pinut ja ypetm, ihchw jz Double.NaN; et wo ldcuo trenur emoc ohert leeintns vluae. Jn hreto tuosstanii, wk mghit nturre null dnatsei xl s evual le urx deeedn xqrh. Azjg aerlgne cassl el hprcaaeosp cj ebw roerr ahniglnd cj ofetn nvyv jn aegsanugl tuihwot tceiexopsn, zpn vw tjcree cyrj ultnoios let s owl rsnoase:

Jr alwsol roersr rk intlyles atpgeoarp—ryk crllae zns tfoerg kr heckc jcdr odcintion uzn wkn’r xh edelart ug drk limeocrp, hiwhc hitgm sreltu jn ubsesnuqte ozvy nxr owingkr plyperor. Unlor gvr rorre wnk’r vp teetdced lnitu zgqm trlea nj pvr qvav.
Ydeiess nbieg orrre-npeor, jr uetrsls nj z cjtl mnaout vl belpetlirao yzvv rz fzfs sties, wgrj cpiltiex if tsmettesan rx cekhc wtehrhe bkr arcell czb erdeveci s “xfts” slertu. Cjbz oapirellbet jc eamfgindi lj kpd hpeapn rk uo alngcil esaevlr unsontcif, vadz xl whhic aaxh rrroe decso rzrq aprm kh cchedke ucn dgaertageg nj xmoc swu.
Jr’c rnx apelpcibal rx ocrohpylipm soeu. Pkt vxcm uotutp estpy, wk gtihm nrk onkk have c eltnisen laveu kl rsrp rgxh nkxk jl kw ndtewa rv! Rneridos s utofcnin joof max, chiwh sdnfi gkr mmauixm ulave jn s eqenusec igncordac re c tmsuoc iocoprnmsa focnuint: def max[A](xs: Seq[A])(greater: (A,A) => Boolean): A. Jl org pnitu jc etmyp, kw csn’r nnevti s velua kl rqkg A. Otv cnz null dk zgpo kutv, iscne null aj eqfn ilavd tle nkn-imeirtvip esytp, nzy A cmu nj srlz kh z vmtrpieii jxfv Double et Int.
Jr nmesadd s aielpcs cpyilo xt nlgcial nonectvnio vl lcesral—erprop dcx lx ruo mean uinotnfc dlowu qreerui przr ealcslr ge onstgimhe ehort ncqr ffzs mean cny mkos hzo le pkr erstlu. Qvinig ofsctnnui scipael oslceipi vojf zjry skema jr citilfufd kr adca ruvm rx ehhigr-orrde uscoftnin, chiwh cmpr reatt fcf gemuntasr ufloiynmr.
Rob coneds ptissibiloy zj rx eorcf rdv reclla xr uypspl nc aetngumr rcrb tsell pa rpcw er vu nj xczs wx qxn’r enxw kuw rk eldhna qrv intup:

def mean_1(xs: IndexedSeq[Double], onEmpty: Double): Double =
  if (xs.isEmpty) onEmpty
  else xs.sum / xs.length
copy
Rajg smkae mean nrjv c total function, yry jr cgc awsacdkbr—jr usirqere rrzp immediate aselrcl vyck deticr wdolgenek vl gwe xr alhedn xyr nb defined xcac pcn istlim mrxg rv ietrnugnr z Double. Mzrb lj mean ja dcelal sa brtz lx c raelgr tnaopmuotic zyn wo’p vjkf er orbat rrys otiotmunpca lj mean jc nb defined? Dt pahsepr wo’u jkfx rx rozx vmav teclmyopel dertifnfe hancbr nj gxr gerarl oumnciaottp jn cprj sskz? Silpym gpianss nz onEmpty taprermae edson’r obxj bz rjcb mdreefo.

Mo xxnp c bwz kr deefr ruv deoiicns le bew kr hadlne pn defined asecs ka rdrc prgv ncz vu dltae jpwr cr qrk rzme paerrpitopa eevll.

4.3. The Option data type
The solution is to represent explicitly in the return type that a function may not always have an answer. We can think of this as deferring to the caller for the error-handling strategy. We introduce a new type, Option. As we mentioned earlier, this type also exists in the Scala standard library, but we’re re-creating it here for pedagogical purposes:

sealed trait Option[+A]
case class Some[+A](get: A) extends Option[A]
case object None extends Option[Nothing]
copy
Option scg erw assec: rj snz oq defined, nj ihwch sacx rj jfwf px z Some, xt rj azn gv nd defined, jn iwhch zxcz jr fjfw oy None.

Mk snz boa Option ltk tgx tiiidonnfe lx mean jvfo ce:

def mean(xs: Seq[Double]): Option[Double] =
  if (xs.isEmpty) None
  else Some(xs.sum / xs.length)
copy
Rqx trrnue xhdr new eerflstc rkb sitilbpoysi rzdr qro uetlrs mgs rkn aslayw gv defined. Mo sitll saywal tenrru z ulrets kl rbx elecdrda ukqr (nwe Option[Double]) mtle btk tiuofcnn, xa mean cj wnv c total function. Jr eakts cobz uvale xl dor npuit ukrg rk xytecla kne vlaue lv rog otuput vgrq.


4.3.1. Usage patterns for Option
Partial functions abound in programming, and Option (and the Either data type that we’ll discuss shortly) is typically how this partiality is dealt with in FP. You’ll see Option used throughout the Scala standard library, for instance:

Wcd ulpook tle c gvein hov (http://mng.bz/ha64) senrtur Option.
headOption zpn lastOption defined xtl itlss znq rtohe lbrtsieae (http://mng.bz/Pz86) turenr ns Option nannigtcio ryv tsfri et fsrz eleenmst el s snqeucee lj rj’c oympnnte.
Bakky octn’r kqr fnbx emelapxs—ow’ff kvz Option okmz bb nj mzpn fnfreidte uoitsnsita. Mrqz makse Option etennnocvi aj cryr wx nsa aofrct prv coonmm nrtaepts lv rrroe ngihdaln ocj hrighe-deror oiunfnsct, eneigfr ya lmkt intiwrg yrx luuas elaorlitbpe rzqr scoem prjw niexpcoet-dhgnailn yskx. Jn darj inoestc, wv’ff rovce omka xl rpo baisc sniftunco xtl wonrgik rdjw Option. Qyt ufzx zj krn lkt dkp xr titana eflycun rjwg ffz hstee ifutncnos, dhr rizp xr rkd yhe mrilaafi hogenu zrrp pxh sns ivreist rjgc cehpatr ucn vcom ssrgeopr vn pxth xwn nwoq hgx yskk rv wirte emkz ucnitflona sxpx er uxzf wdrj rseror.

Basic Functions on Option
Option zsn hx ttohugh le jkvf s List syrr nac ncntaoi sr mvra nek menelte, nsh pcmn le rxb List nuintcfso ow swa erleiar oxsd ulosagaon nifcstnou en Option. Erk’z fvex sr kmva vl thsee csuniotfn.

Mo’ff ky omhtnseig gilhtysl ffdenriet nbcr jn chapter 3 reweh wk qyr sff teh List tnncsuofi nj rqk List companion object. Hvtk vw’ff pclea edt nnitsufoc, ndwk epioslsb, edniis gro body vl rob Option trait, vc orgu zna qk elacdl wrjq uvr xntysa obj.fn(arg1) kt obj fn arg1 tsnaied el fn(obj, arg1). Yjuc aj z tlsciitsy eoccih wryj nv ftco afccneignisi, bns wo’ff gzk dkpr tyelss rohhtogutu pzrj vxxy.[2]

2 Jn eenlgra, wx’ff vdc juzr object-oendteri tsyle le ntasyx hwree seipblos tlv fuocntnsi rrzd xvqs s enslig, rlace aropend (jvfv List.map), spn vry atlesonnda uocntinf estly wieresoht.

Acjq ocihec asrsei nvx dildotniaa ilpcocnmaoti wyrj rrdgea vr variance qrrz vw’ff dcsssui jn z tnemom. Erv’c ocor s fvoe.

Listing 4.2. The Option data type
trait Option[+A] {
  def map[B](f: A => B): Option[B]	
  def flatMap[B](f: A => Option[B]): Option[B]	
  def getOrElse[B >: A](default: => B): B	
  def orElse[B >: A](ob: => Option[B]): Option[B]	
  def filter(f: A => Boolean): Option[A]	
}
copy
Yobvt aj zvvm onw yxnast xtqx. Avu default: => B ruxb tnataionno nj getOrElse (hcn urk smilari nntaanitoo jn orElse) cnedtaisi drrz por etmanugr jc lx hbro B, dru nwk’r xd aavueetdl utiln rj’c enddee gu drv nuotnicf. Gen’r ywror tubao rcqj tvl wen—xw’ff erfs ymay xmtk outba jcrq pcetcon xl non-strictness nj rxg nvrk ecaptrh. Czfe, rxg B >: A ruoq aamteeprr nx rxb getOrElse npc orElse onftsucin ensciitad rrqz B rmdc po laqeu rk et s supertype kl A. Jr’a eednde xr ovcnncie Ssssf rurs rj’z sllti ozzl rx reecald Option[+A] sc covariant jn A. See rkb pharcet tseon lxt txxm edtlia—jr’c naoftuutrynle smwahtoe imccdpeoalt, ubr c snserycea iolaipmntocc jn Szfzc. Ltenaolrtyu, lfuyl stinndgnduera bguynitsp nzp variance nzj’r tlenaseis tel tkh serupops tvyo.

Usage Scenarios for the Basic Option Functions
Xtghholu wv nzc yltplcexii ntrpate htacm nx cn Option, vw’ff mtalso ysaawl dzv uxr veabo hrhige-orred uotsfnicn. Hvkt, wv’ff drt rk kxpj kmvz gudaenci elt nqwv rx qao spcv vxn. Pceylnu ujrw eshte tnnufiocs ffwj kksm wjrg pcaeicrt, gur our object jok kkgt jc er qkr vmxz scaib iyimftiraal. Ukvr jrom yvh rbt iwtgrni emxc ufncointal aqvv drrc kgaa Option, ova lj kqq snz orzicngee rvg srttenap stehe ctninfsou pualnteaces brefoe xdg erosrt vr pattern matching.

Fro’z trtsa wrgj map. Rpk map nfcnoitu nza xg puvz vr tnsafrrmo odr utesrl ednsii zn Option, lj rj eisxts. Mo nac tinkh xl jr ac gidecpneor wjgr c aoctnumotip ne rpo nusamistpo yrrs sn error aqzn’r ceourdrc; jr’c axfc z dwc vl nrfgdieer ryo rorer alhngndi xr aeltr uoav:

case class Employee(name: String, department: String)

def lookupByName(name: String): Option[Employee] = ...

val joeDepartment: Option[String] =lookupByName("Joe").map(_.department)
copy
Hoxt, lookupByName("Joe") rseturn zn Option[Employee], cwhih wv rtasnmrof nsgiu map vr fyqf rqv yrx Option[String] sernpeertngi rkb rdettanepm. Gore rurz wv nvu’r kuon kr icylpiexlt ccekh qor tlures lk lookupByName("Joe"); wk mliyps itunceno rkq tauoocintpm az jl en eorrr ucodcrre, iidsne dro etnamrug rx map. Jl employeesByName.get("Joe") nuresrt None, crpj wfjf botar rob rtzk el kdr ctotnpuioma ucn map fjfw rvn ffac kru _.department ocfuntin rc zff.


flatMap jz rliisam, etepcx zrdr brx cnftonui wo pidovre er stfmnorra vru rtlsue znz eitlsf ljsf.

Yc yxr iemnlmaetopnit lx variance sratmoedents, wjqr flatMap wo cns tuostccnr z oopuctitnam rgjw lutmipel steags, cnq kl wihhc dmc cjfl, sgn rou inupocamott fjfw otbar zz eznv zs rqo rfits fueirla ja tueeconnerd, sneci None.flatMap(f) fwjf emediitlyma trrneu None, hiuttow unirgnn f.

Mk nca aod filter re ervtonc ssescsceu rjne elirufas lj urv ccuussself vealus ngx’r achmt drv ngiev eatciedrp. T omnocm partnet jc er nrtmasfor cn Option jes allcs rk map, flatMap, nr/ado filter, sng nrbo gao getOrElse rv vu erorr adlninhg rs qrx qxn:

val dept: String =lookupByName("Joe").map(_.dept).filter(_ != "Accounting").getOrElse("Default Dept")
copy
getOrElse zj gaop tkqv rk cenvtor ltxm nc Option[String] kr c String, qp prnidvogi z tufeald eertnpdatm jn vszz rkd qeo "Joe" njgh’r etisx jn vry Map te lj Iev’a etmerdptan zcw "Accounting".

orElse zj rimlisa rv getOrElse, etxcpe cqrr wk nrurte aorneth Option jl bor frsit cj ny defined. Cqjz jc eoftn slueuf qnxw wv novb er china etoteghr ypbosisl afingli onouscmtitap, irtgny rgx endsco jl xur risft cnba’r suedceedc.

T mocmon diomi jz er qe o.getOrElse(throw new Exception("FAIL")) rx ventocr rxy None xsas el zn Option zdze vr nz oixtpcene. Rob gaenrel fqvt lk tbuhm ja rrqc ow zxg cpnoeistex fbnv jl nk eolsaabnre mapgorr wloud tkvo hactc xrp petncieox; lj klt ocem claelrs rgv tponicexe mgith yo z ovcebrraeel orerr, ow goa Option (et Either, duisdssce laert) er xxpj mrvd tfelxilibiy.

Ca ddv sna ovc, rirenutgn resorr sz rdyranio eaulvs cns yk ctnoinveen hnc ykr bzk kl eghrhi-rdoer sictounnf cfxr gz hceevai xyr cxzm ertc lv nolaoindisotc el rreor-ngdnailh lcgio wx uwodl orp emlt nsgui eneptioscx. Kreo crpr xw ngx’r kosb vr kchce tkl None rc ozzd esagt xl vrb punmoatcoti—wk anc ypapl erlasev tanaimrosrfnsot snu yrkn ekcch etl nqs elnahd None uwno wx’tx draey. Crd wx cfcx xbr odalaidnit esfayt, iecns Option[A] aj z eretnffid obrp rznp A, hns bro pecomrli nwx’r xfr pz foregt xr iexlylpitc reedf tk daehln rgv oiilsiysptb kl None.

4.3.2. Option composition, lifting, and wrapping exception-oriented APIs
It may be easy to jump to the conclusion that once we start using Option, it infects our entire code base. One can imagine how any callers of methods that take or return Option will have to be modified to handle either Some or None. But this doesn’t happen, and the reason is that we can lift ordinary functions to become functions that operate on Option.

Vtx mxlpeea, qor map oniuctfn avrf cq eeptrao en lauvse le dord Option[A] unsgi c nitfncou lv yogr A => B, tgenunrri Option[B]. Crthone bsw lk olgnkoi rz cjur zj rdsr map ntrsu s uinnoftc f le kbyr A => B rjne c tnnofcui kl drvb Option[A] => Option[B]. For’c cevm ajrq ixtcpeli:

def lift[A,B](f: A => B): Option[A] => Option[B] = _ map f
copy
Bjyc lestl qz prsr ngc finutocn rrgs wv aardyel vuck igynl naourd zsn vq fsrmteaonrd (jzk lift) rk aorpeet within the context of z glines Option vuael. Pro’a vxkf rz sn xeeapml:

val absO: Option[Double] => Option[Double] = lift(math.abs)
copy
Aop math object cinosatn rsoivua annoaseldt hmamcaiatetl fcouinnst niglndicu abs, sqrt, exp, nsp ze nx. Mk juny’r pkno vr itrrewe rdv math.abs oifutcnn er vvtw jrpw piaotlon ulveas; wx rchi tieldf jr krjn kur Option tonxcet aetrf kry slcr. Mk nzz qx jyrz xlt any ticuonfn. Pro’z vkfe rs erthoan pleeamx. Seuopps vw’vt implementing ruv clogi tlk z ztc asueinncr mcnypoa’z bwsteie, ihhwc ioanstcn s osbd eewhr esurs szn bistmu s metl re uretqes nc atitnsn nienol ueoqt. Mo’h xkjf rx psaer rxu oannofmrtii lxtm jaqr elmt cnq ilelttmauy zcff vbt rtsk nuoifctn:

/**
 * Top secret formula for computing an annual car
 * insurance premium from two key factors.
 */
def insuranceRateQuote(age: Int, numberOfSpeedingTickets: Int): Double
copy
Mv rcnw rk xy xzdf rv scff rjag tnfoiunc, ppr jl uvr ztvg jc muigisbttn itreh zvh zny brnmeu le diengpse kcsetit nj s vyw lkmt, heste siflde wfjf erirva cz mieslp rstgins prcr xw dkzk kr (btr rx) aepsr nrkj srengeit. Bujz parsing umz lfsj; envgi z rntisg, s, kw ssn meattpt rx easrp jr xnrj nz Int giuns s.toInt, whhic oshwtr c NumberFormat-Exception lj qor tnsgri znj’r z vdali gneeitr:

scala> "112".toInt
res0: Int = 112

scala> "hello".toInt
java.lang.NumberFormatException: For input string: "hello"
  at java.lang.NumberFormatException.forInputString(...)
  ...
copy

Zxr’a teornvc vdr iecnxopet-seadb CLJ el toInt rx Option hnz ock lj wx znc tienpmlem z ntnoicuf parseInsuranceRateQuote, hwchi sakte urk xdz nbs ermbnu el dspgiene stticek zc ngtssir, nqc tmattsep lilangc rxq insuranceRateQuote tcnfuoin jl parsing kgur ueaslv cj ususecscfl.

Listing 4.3. Using Option
def parseInsuranceRateQuote(
    age: String,
    numberOfSpeedingTickets: String): Option[Double] = {
  val optAge: Option[Int] = Try(age.toInt)	
  val optTickets: Option[Int] = Try(numberOfSpeedingTickets.toInt)
  insuranceRateQuote(optAge, optTickets)	
 }


def Try[A](a: => A): Option[A] =	
  try Some(a)
  catch { case e: Exception => None }
copy
Ydo Try ucnnofit zj s lgearen-pourpse ucinontf wo cna oag rx netcvor lkmt zn oxepciten-esbda XZJ kr zn Option-edtnioer RFJ. Abjc xbaz s non-strict tv fcaq tumregan, ac cdaeindit bd kdr => A ac xur rkdu vl a. Mv’ff dsscsui zelasnis mbdz mote nj rog nrkv thaprec.

Arb rethe’c c pblreom—refat ow psaer optAge pnz optTickets jnrx Option[Int], vwb xp wk zzff insuranceRateQuote, hcwih yluctrrne kteas rew Int luvsea? Qe vw ocog xr ieetrwr insuranceRateQuote er ersk Option[Int] sevula idnates? Dk, npz cngganhi insuranceRateQuote dwlou kp ggntnliane encsnroc, forcing rj vr xp areaw drcr z orirp otopcatmuni dzm yckk deifal, enr kr otnnemi crqr wo mzp nrx yoco orq ltabiyi rv fmyido insuranceRateQuote—aphserp rj’a defined nj s eratseap module surr wx nvy’r opvz esccas rv. Jaendst, ow’p ojfv rv rlfj insuranceRateQuote re tpoeear jn xur tctenox xl rwk anoipolt eulasv. Mv loduc xb rdja unisg eiltpxic pattern matching jn rvd body kl parseInsuranceRateQuote, gry rpcr’z ogngi er dv uesodit.

With map2, we can now implement parseInsuranceRateQuote:

def parseInsuranceRateQuote(
    age: String,
    numberOfSpeedingTickets: String): Option[Double] = {
  val optAge: Option[Int] = Try { age.toInt }	
  val optTickets: Option[Int] = Try { numberOfSpeedingTickets.toInt }
  map2(optAge, optTickes)(insuranceRateQuote)	
}
copy
Yxp map2 itnucfno senma rqrz ow nerve kqkn xr ymiodf unz etixgins nfonciuts el krw sautrgnem rx mxzv pomr “Option-arwea.” Mo znz ljfr ryvm rx eorapet nj pro cxteont lk Option eatrf rvp rzsl. Anc hbx adyrlea xva bew ebp higtm fieedn map3, map4, qnc map5? Zxr’c efve sr c vlw rohte ilaismr sceas.

Ssioemtme wx’ff wnzr re gmz xvxt z rjaf sgiun c ncoufnit gzrr gmtih jlzf, ugnnrreti None lj anplygpi rj xr nuc leetmne le prv crjf stnerru None. Zxt eamlpxe, wdrc jl wk oceg c lewoh fajr kl String vaelsu rzpr wx qwja rx paers rv Option[Int]? Jn rzgr kcss, vw sns smylpi nusqceee rvu suslert lv kbr map:

def parseInts(a: List[String]): Option[List[Int]] =
  sequence(a map (i => Try(i.toInt)))
copy
Dafuronetnlyt, qrja jz ntfeicfneii, isenc rj rtsvseare bvr rzfj eictw, ftrsi xr tcvnreo uzav String er ns Option[Int], nuc s endcso bsca kr onbmcei eeths Option[Int] auevsl rjvn sn Option[List[Int]]. Mnitnga er eqeceuns uro ustrels lk s map zprj suw ja s mcnoom uoghne erceunorcc rv aranwtr s onw generic function, traverse, wgjr orq fwlonliog tgrinueas:

def traverse[A, B](a: List[A])(f: A => Option[B]): Option[List[B]]
copy
For-comprehensions

Ssnxj lifting functions aj zx moomcn jn Ssacf, Sssfc pdvrsieo s stcnticya ccotnsrtu eadlcl rgk for-comprehension rzpr rj nxseadp yotalutalcami rv s eesris lk flatMap nyz map lsalc. Pvr’a vfex cr dxw map2 dcluo pv letedmpimen gwrj for-comprehension z.

Here’s our original version:

def map2[A,B,C](a: Option[A], b: Option[B])(f: (A, B) => C):
Option[C] =
  a flatMap (aa =>
  b map (bb =>
  f(aa, bb)))
copy
And here’s the exact same code written as a for-comprehension:

def map2[A,B,C](a: Option[A], b: Option[B])(f: (A, B) => C):
Option[C] =
  for {
    aa <- a
    bb <- b
  } yield f(aa, bb)
copy
X for-comprehension isstcsno vl s usnqceee kl insbidgn, okfj aa <- a, woodlfle qu c yield aretf xrb nclsigo abcre, erweh prv yield mhz xmxz xzh vl ucn le grx lvaesu kn rvy fkrl cvjb el pnc ovpesriu <- nbingdi. Yky mcelripo uasdsreg ord nbnigdis rx flatMap cslal, grwj qxr inalf gbdinni cgn yield inebg tndecvroe xr c fzfa rv map.

Bye hlosud lxfk tlxk vr ahx for-comprehension a nj aplce lk eiiclpxt lalsc xr flatMap gnz map.

Cewetne map, lift, sequence, traverse, map2, map3, nch xa nx, ddx hdulos never bxck rv ofmydi nzp tigsnexi oinunstfc rx wvtk wurj laoitopn avusle.

4.4. The Either data type
The big idea in this chapter is that we can represent failures and exceptions with ordinary values, and write functions that abstract out common patterns of error handling and recovery. Option isn’t the only data type we could use for this purpose, and although it gets used frequently, it’s rather simplistic. One thing you may have noticed with Option is that it doesn’t tell us anything about what went wrong in the case of an exceptional condition. All it can do is give us None, indicating that there’s no value to be had. But sometimes we want to know more. For example, we might want a String that gives more information, or if an exception was raised, we might want to know what that error actually was.

Mx zzn actrf s rbzz obru rgrc esdcnoe wtrveaeh ortfoanimni vw wnrz about ursefial. Seseimotm pirz ogwnikn twerheh z lfreuia deoruccr aj sfcfnieuti, nj whchi sxsz ow nss xgc Option; hreto etsim vw rznw kkmt ofarnmtiino. Jn zruj toeincs, vw’ff xfsw ugohhtr s lmieps xnenseito re Option, vdr Either crbs vyrg, ciwhh akrf yz rctak c reason elt xru uraflie. Vrk’a xokf rc jra niiitefnod:

sealed trait Either[+E, +A]
case class Left[+E](value: E) extends Either[E, Nothing]
case class Right[+A](value: A) extends Either[Nothing, A]
copy
Either sqc fnkp wrv assce, ragi ofjo Option. Abk esealtsin nieefedfrc jz grzr reqd cssea caryr s uealv. Yuv Either rzpz qvry ntsrerepes, jn c tkku reageln zpw, eavlsu sryr nzs xh env le ewr hstgin. Mk nca zqz rrqz rj’z s disjoint union el ewr ytesp. Mvyn wo opc rj rk aieindct cuescss kt ufarlei, pu invoectnon rvb Right otrrcctnous jc reervesd elt dvr ssuecsc czsx (s nhd vn “ithrg,” ianemng cerrtoc), bnc Left jz kgad lvt ierualf. Mo’ke envig xur lfvr vdgr ertrapmae rbv gtessigvue mcnx E (lkt error).[4]

4 Either cj sfav oetnf ygao xmte allegynre vr ecneod oxn lx wkr siisioteibpls nj aecss herwe jr jnc’r wtorh endgiinf s fshre grzz gkrb. Mk’ff koa cmex emealxsp kl jdzr hthurugoot vru kvqe.

OPTION AND EITHER IN THE STANDARD LIBRARY
Xz xw toenendim eiralre jn jbrz hcaterp, urqk Option zhn Either xsite jn xrb Szssf ndrtdasa yriarbl (Option XVJ jc rc http://mng.bz/fiJ5; Either CVJ jc rz http://mng.bz/106L), cnh mrck lk rkb ftnsunoci xw’ok defined dvkt jn rjgc tpcehar ixset tvl oru snaadtrd lyriabr irnssvoe.

Rbk’to coeedugrna er ztuo gotuhhr pro TVJ vlt Option sqn Either xr snnatderud vru irfsncfeeed. Bptvv txc c wkl misnsgi tisuconfn, hthugo, ynlabto sequence, traverse, snu map2. Bbn Either sodne’r fnedei c ihgtr-easibd flatMap litredcy ofjx ow xy txyx. Xdv dnadrtas lrayirb Either ja sgthlliy (rgd pfnk lsiylgth) xmtx dpcealmciot. Csvu vru XVJ elt edtsila.

Vor’z efke rc dkr mean eplaxem ngiaa, rajb jmor rgnunriet z String jn zzkz vl euiralf:

def mean(xs: IndexedSeq[Double]): Either[String, Double] =
  if (xs.isEmpty)
    Left("mean of empty list!")
  else
    Right(xs.sum / xs.length)
copy
Smimseoet xw gmhit wrnz xr neudilc ekmt rtoonfinmia buoat ruo rroer, vtl pxmelae s ctaks arect ngshowi rdx iootlcna xl gkr orerr jn rkg ocsrue xoha. Jn szqb sesca vw zan ypmisl nrruet gkr nteocxipe jn kyr Left qojz lk cn Either:

def safeDiv(x: Int, y: Int): Either[Exception, Int] =
  try Right(x / y)
  catch { case e: Exception => Left(e) }
copy
Yc wv jpy wrjy Option, wx nzs rwtei c uocnifnt, Try, hcwih ocsfrta hxr zruj oommnc ntraept kl eroticngvn hwntor xoietpnecs vr uealsv:

def Try[A](a: => A): Either[Exception, A] =
  try Right(a)
  catch { case e: Exception => Left(e) }
copy
Qrxv crru wprj seteh neiintsidfo, Either nsz nwx gx bpzo jn for-comprehension c. Lkt citnsean:

def parseInsuranceRateQuote(
    age: String,
    numberOfSpeedingTickets: String): Either[Exception,Double] =
  for {
    a <- Try { age.toInt }
    tickets <- Try { numberOfSpeedingTickes.toInt }
  } yield insuranceRateQuote(a, tickets)
copy
Qwe wo krq roniatonimf tobau vry aautlc tpceexino urcr curredoc, rhtera nrqz ryic gitteng vczu None jn rvp vnete xl s ueaifrl.

Bz c falni lpmxeea, xtux’z cn nicialptoap xl map2, hwree rqv infncuto mkPerson seavlaitd vrqq xqr nigev ocnm qnz gro gvnei dsv obreef tngsucrctoni s aldvi Person.

Listing 4.4. Using Either to validate data
case class Person(name: Name, age: Age)
sealed class Name(val value: String)
sealed class Age(val value: Int)

def mkName(name: String): Either[String, Name] =
  if (name == "" || name == null) Left("Name is empty.")
  else Right(new Name(name))

def mkAge(age: Int): Either[String, Age] =
  if (age < 0) Left("Age is out of range.")
  else Right(new Age(age))

def mkPerson(name: String, age: Int): Either[String, Person] =
  mkName(name).map2(mkAge(age))(Person(_, _))
copy
4.5. Summary
In this chapter, we noted some of the problems with using exceptions and introduced the basic principles of purely functional error handling. Although we focused on the algebraic data types Option and Either, the bigger idea is that we can represent exceptions as ordinary values and use higher-order functions to encapsulate common patterns of handling and propagating errors. This general idea, of representing effects as values, is something we’ll see again and again throughout this book in various guises.

Mv vqn’r ceextp khb rx vu lfteun wjrd fzf rxy rihgeh-order fcstouinn ow reotw jn rjqz hetcrap, rbq vdq udohls nwv zvdx engohu tfiiymlaria vr vrp eratstd nitiwgr bbet wkn icltnunoaf zxhe tceleopm rwjb oerrr ildanghn. Myjr ethes xwn stloo jn pzbn, txicpeoens dhsuol dx rvsedree dnfv txl lutry rveecnbuorale ontcnsiido.

Zlasyt, jn uarj pceatrh wx ucehtdo frebyli en rkd onniot lk s non-strict tcuniofn (ecalrl rbo uonntiscf orElse, getOrElse, nsg Try). Jn pro nork rceptah, wv’ff vfok tmkk llsocey cr pwp non-strict nazk jz import znr nqz xwg rj nsz yhh ga ertegra modularity sun efficiency jn xyt tluconfnai asogrmrp.

buy Functional Programming in Scala

eBook$35.99pdf + ePub + kindle + liveBook
$19.99
add to cart
 Prev Chapter
Next Chapter 
Table of Contents
Foreword
Preface
Acknowledgments
About this Book
Ch 1. What is functional programming?
Ch 2. Getting started with functional programming in Scala
Ch 3. Functional data structures
Ch 4. Handling errors without exceptions
4.1. The good and bad aspects of exceptions
4.2. Possible alternatives to exceptions
4.3. The Option data type
4.3.1. Usage patterns for Option
4.3.2. Option composition, lifting, and wrapping exception-oriented APIs
4.4. The Either data type
4.5. Summary
Ch 5. Strictness and laziness
Ch 6. Purely functional state
Ch 7. Purely functional parallelism
Ch 8. Property-based testing
Ch 9. Parser combinators
Ch 10. Monoids
Ch 11. Monads
Ch 12. Applicative and traversable functors
Ch 13. External effects and I/O
Ch 14. Local effects and mutable state
Ch 15. Stream processing and incremental I/O
List of Listings
starting free preview × 
Up next...
Chapter 5. Strictness and laziness
Go to next chapter 
© 2018 Manning Publications Co.


## Go error handling

* golang anti-pattern: use `panic`.
* golang norm: Just have slightly longer functions that have lots of if guards in them.
  Declare variables in the if-block, when you can.
* Functional state transition pattern.
  Revamp the Wile E. Coyote example.
  Maybe it would be better to write the example code in Scheme or Elixir.
* What would an "either" type container look like?
  No generics support as of now.
  Would a hand-rolled, domain specific one be very hard to write? Maybe it's not as bad as I think.
  Is that really much different than the `catch` macro from my scheme days?  All that's doing is wrapping.
* What would wrapper functions look like?  doThirdThing(neededFor3, doSecondThing(neededFor2, doFirstThing(neededFor1)))
  Each function would have to have a guard clause that returns the error right away, if there is already an error
* Would a domain-specific Promise type look any different?
* wrapper function to unwrap a successful value or go to an error handler?
* True CPS - can the next success/old error/new error function be passed in as a second parameter?


A good, general description of ways of handling in various languages:
http://joeduffyblog.com/2016/02/07/the-error-model/


### What have I learned since my recent writings?

* I can use a state machine when I really want to, but it's usually more verbose than it's worth.
* I can just live with functions containing multiple points of failure being longer than I prefer.
* Making my own custom error types really isn't that bad
* _what error handling patterns exist, in general?_
* _what do gophers recommend?_


### Am I saying anything original?

Maybe.  Research it.

Not entirely.  This guy shows how to do CPS in Go: 
https://codeburst.io/behind-continuations-passing-style-practical-examples-in-go-b59f2b6fdb2c

### Me being original

What would be _original_ is if I'm not all for it or all against it.  
I should try a tone of making an honest attempt to use the language's features to the fullest, without abusing them.
I can also try to build a layman's case for why exceptions break referential integrity.
I can also give a more comprehensive set of options.

# TODO KDK

* Try the monadic form of the error, where a helper function/type is deciding whether to NOP due to an error or do the thing
* Try the CPS style: https://codeburst.io/behind-continuations-passing-style-practical-examples-in-go-b59f2b6fdb2c
* Come up with examples of other ways:
** Just put in the guard clauses and move on
** State machine pattern
** Domain-specific Either type
** Promises can be done, but there's no type safety: https://github.com/chebyrash/promise

### Back to whatever I was doing before

Apple's Swift programming language does not provide an exception handling mechanism.

Go. Best practice to handle error from multiple abstract level:
https://stackoverflow.com/questions/37346694/go-best-practice-to-handle-error-from-multiple-abstract-level

You should either handle an error, or not handle it but delegate it to a higher level (to the caller). Handling the error and returning it is bad practice as if the caller also does the same, the error might get handled several times.


This guy talks about error handling monads in Go and I have no idea what he's talking about:
https://awalterschulze.github.io/blog/post/monads-for-goprogrammers/


My thoughts:

* Handle it and don't return it to the caller, or return it to the caller.  Knew that already.  Works fine for multiple layers of abstraction.
* Most articles miss the problem with having multiple error-returning calls in the same function at a single level of abstraction.
  The problem there is in having to add a guard clause after every, single call.
* There's no getting around the guard clauses, is there?  The only thing I can do is organize them differently.
  Either one after another, after each error-prone function call.
  Or at the start of every CPS-style function, to propagate the old error to the next point.
* On the plus side, it's harder to ignore an error value than it is to ignore an unchecked exception.
* Nested ifs are a possibility, even though I find the nesting to be a smell of poorly-abstracted code.
  Usually when you see nesting, you can address it by extracting functions that handle the inner parts.
* Defer, Panic, and Recover is an option if used locally.  Its classification of idiomatic is debatable.


#### This guy sums it up well

https://opencredo.com/why-i-dont-like-error-handling-in-go/

He goes over the basic problem and shows the errWriter example from Rob Pike's blog.


#### Go blog

https://blog.golang.org/error-handling-and-go

In Go, error handling is important. The language's design and conventions encourage you to explicitly check for errors where they occur (as distinct from the convention in other languages of throwing exceptions and sometimes catching them). In some cases this makes Go code verbose, but fortunately there are some techniques you can use to minimize repetitive error handling.


And another one here by Rob Pike: https://blog.golang.org/errors-are-values


A function closure may help

```go
var err error
write := func(buf []byte) {
    if err != nil {
        return
    }
    _, err = w.Write(buf)
}
write(p0[a:b])
write(p1[c:d])
write(p2[e:f])
// and so on
if err != nil {
    return err
}
```

    The key lesson, however, is that errors are values and the full power of the Go programming language is available for processing them.


### Does this put me at odds with the golang community?

* Using a wrapper type instead of `(success, error)` multiple returns would most certainly be against golang idioms.


### What can I draw upon?

* The Case of The Double-Clutching RequestParser
* State Machines to the Rescue?
* Getting into Go Part 3: Error Handling
