################################################################################
##                                                                            ##
##  This file is part of ModelBlocks. Copyright 2009, ModelBlocks developers. ##
##                                                                            ##
##  ModelBlocks is free software: you can redistribute it and/or modify       ##
##  it under the terms of the GNU General Public License as published by      ##
##  the Free Software Foundation, either version 3 of the License, or         ##
##  (at your option) any later version.                                       ##
##                                                                            ##
##  ModelBlocks is distributed in the hope that it will be useful,            ##
##  but WITHOUT ANY WARRANTY; without even the implied warranty of            ##
##  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the             ##
##  GNU General Public License for more details.                              ##
##                                                                            ##
##  You should have received a copy of the GNU General Public License         ##
##  along with ModelBlocks.  If not, see <http://www.gnu.org/licenses/>.      ##
##                                                                            ##
################################################################################

################################################################################
# 
#  This directory contains reusable recipes for setting up reading time studies.
#  The recipes produce and then merge data from:
#  
#  1. %.pcfg.tokmeasures: Incremental pcfg parser output (parser-tokeniziation - 
#     separate punctuation toks)
#  2. %.syn.tokmeasures: PCFG praser output with center-embedding data 
#     calculated from gold trees (parser tokenization)
#  3. %.itemmeasures: All syntactic predictors with ngram probabilities 
#     (experiment tokenization, in which parser tokenization should be nested)
#  4. %.full.evmeasures: itemmeasures data merged with reading event data 
#     (*.src.evmeasures), i.e. the complete data set
#
#  To generate these files, the experiment must have (or have recipes to 
#  generate) the following intermediate files:
#  
#  1. %.linetoks: Parser-tokenized file of space separate sentences, one on each
#     line
#  2. %.lineitems: Experiment-tokenized file of space separated sentences,
#     one on each line
#  3. %.src.evmeasures: Space-delimited table of reading event data from source
#     experiment (IMPORTANT: this must include sentid and sentpos columns)
#  4. Params files:
#     a. <corpus_basename>.merge_tables.params: merge key fields (and other
#        params if needed) to pass to merge_tables.py. Typically
#        'word sentid sentpos'
#     b. <corpus_basename>.accumulateMetrics.params: fields to accumulate
#        (and other params) to pass to accumulateMetrics.py
#     c. <corpus_basename>.rm_bad_toks.py: params to pass to rm_bad_toks.py
#
#  For working recipes to generate each of these kinds of files, refer to
#  resource-dundee/Makefile.
#
################################################################################

################################################################################
#
#  Includes
#
#  Include statements to all makefiles necessary to build experiment-specific
#  prerequisites to these recipes should precede the include statement for
#  this makefile.
#  
################################################################################

################################################################################
#
#  Macros & variables
#
################################################################################

.SUFFIXES:
SECONDEXPANSION:

MERGESURP = python $(word 1, $^) --input $(subst GRAMMAR,$(grammar),$(basename $*)).full.evmeasures --i totsurp | \
sed '1s/.*/$(subst -,,$(grammar))totsurp/' | paste -d' ' $@ - > $@.tmp; \
mv $@{.tmp,}; \
python $(word 1, $^) --input $(subst GRAMMAR,$(grammar),$(basename $*)).full.evmeasures --i cumtotsurp | \
sed '1s/.*/cum$(subst -,,$(grammar))totsurp/' | paste -d' ' $@ - > $@.tmp; \
mv $@{.tmp,};

SPILLOVER := S1 S2 S3
DLT := dlt dltc dltcv dltv dltm dltcm dltcvm dltvm
FJ := fjprob noFprob endembd embdlen embdlendr embdlendrv noF noFlen noFlendr noFlendrv reinst reinstlen reinstlendr reinstlendrv
EXTRA-SPILL := dc dcv pos depdirSyn depdirSynM depdirSem depdirSemM Ad AdPrim Bd BdPrim Bdm1 Bdm1Prim noFprob embddepth embdif
GCG-CATS := A B C D F G
EMBDDEPTHS := 1 2 3 4 5

# The following macros automate generation of a lot of useful latency experiment
# configurations using the following template:
#
#     <experiment-name>-[cum]-[<spillover-shift>]_[<lmeargs>]
#
# Supported experiment names include all effect names in variables FJ and DLT above,
# as well as FJ (all effects stored in variable FJ), DLT (all effects stored in variable
# DLT), and MEM (all effects stored in variables FJ and DLT together). When included,
# the -cum- parameter runs the variant of the experiment with predictor(s) accumulated
# over saccade regions. When included, the -<spillover-shift>- parameter runs the
# experiment using the nth-degree spillover variant of the predictor(s) (currently S1,
# S2, and S3 supported). The optional _<lmeargs> parameter passes custom command line
# options to the main experiment script, resource-lmefit/scripts/evmeasures2lmefit.r.
# For documentation of these, run:
#
#     ../resource-lmefit/scripts/evmeasures2lmefit.r -h
#
# Examples:
#
#     1) MEM-NSCFl
#     2) DLT-S3-NSCFILl
#     3) fjprob fjprob-S1 fjprob-cum fjprob-cum-S1
#
# (1) runs all dlt and fj predictors using LME options "-NSCFl".
# (2) runs all third-degree spillover dlt predictors using LME options "-NSCFILl".
# (3) runs basic and 1st-degree spillover variants of accumulated and non-
#     accumulated variants of the fjprob predictor, using (locally-defined)
#     default LME options.
#
# To make these recipes available in a client makefile, simply add the line:
#
#     $(eval $(RT-EXPERIMENTS))
#
# Note: client makefiles must locally define the variables BASENAME and LMEDEFAULT,
# where BASENAME is the string preceding the main effect name in the *.lrtsignif
# target (see e.g. dundee/Makefile for example) and LMEDEFAULT are the default
# command line options to pass to resource-lmefit/scripts/evmeasures2lmefit.r
# (e.g. "NSCF").
#
# Also note: these recipes will not work unless the client makefile also includes
# $(RESOURCE-RT)/Makefile and all of its dependencies.
#
# Also also note: LME regressions are quite memory-intensive (usually requiring
# 2-3 times the size of your data table). Each pairwise anova created by the
# recipes below will require two such regressions. Keep this in mind when
# using parallel make (make -j), since it's easy to generate OOM errors.
# For example, KITCHENSINK below runs (|FJ|+|DLT|)*(|SPILLOVER|+1)*2*2 = 208
# total regressions, which will exceed the memory capacity of most systems.
# When running in parallel, make sure to hard-limit the number of parallel
# processes to a reasonable level (e.g. make -j10 KITCHENSINK).
#
define PRINTL
$(1)
endef

define SPILL
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-$(1):$(BASENAME).$(var)$(1).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-$(1)_%:$(BASENAME).$(var)$(1).-%..lrtsignif;)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-$(1):$(BASENAME).$(var)$(1).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-$(1)_%:$(BASENAME).$(var)$(1).-%..lrtsignif;)))
DLT-$(1): $(foreach var, $(DLT), $(var)-$(1))
DLT-$(1)_%: $(foreach var, $(DLT), $(var)-$(1)_%);
FJ-$(1): $(foreach var, $(FJ), $(var)-$(1))
FJ-$(1)_%: $(foreach var, $(FJ), $(var)-$(1)_%);
MEM-$(1): DLT-$(1) FJ-$(1)
MEM-$(1)_%: DLT-$(1)_% FJ-$(1)_%;

$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum-$(1):$(BASENAME).cum$(var)$(1).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum-$(1)_%:$(BASENAME).cum$(var)$(1).-%..lrtsignif;)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum-$(1):$(BASENAME).cum$(var)$(1).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum-$(1)_%:$(BASENAME).cum$(var)$(1).-%..lrtsignif;)))
DLT-cum-$(1): $(foreach var, $(DLT), $(var)-cum-$(1))
DLT-cum-$(1)_%: $(foreach var, $(DLT), $(var)-cum-$(1)_%);
FJ-cum-$(1): $(foreach var, $(FJ), $(var)-cum-$(1))
FJ-cum-$(1)_%: $(foreach var, $(FJ), $(var)-cum-$(1)_%);
MEM-cum-$(1): DLT-cum-$(1) FJ-cum-$(1)
MEM-cum-$(1)_%: DLT-cum-$(1)_% FJ-cum-$(1)_%;
endef

define RT-EXPERIMENTS
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var):$(BASENAME).$(var).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)_%:$(BASENAME).$(var).-%..lrtsignif;)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var):$(BASENAME).$(var).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)_%:$(BASENAME).$(var).-%..lrtsignif;)))
DLT: $(DLT)
DLT_%: $(foreach var, $(DLT), $(var)_%);
FJ: $(FJ)
FJ_%: $(foreach var, $(FJ), $(var)_%);
MEM: DLT FJ
MEM_%: DLT_% FJ_%;

$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum:$(BASENAME).cum$(var).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum_%:$(BASENAME).cum$(var).-%..lrtsignif;)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum:$(BASENAME).cum$(var).-$(LMEDEFAULT)..lrtsignif)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum_%:$(BASENAME).cum$(var).-%..lrtsignif;)))
DLT-cum: $(foreach var, $(DLT), $(var)-cum)
DLT-cum_%: $(foreach var, $(DLT), $(var)-cum_%);
FJ-cum: $(foreach var, $(FJ), $(var)-cum)
FJ-cum_%: $(foreach var, $(FJ), $(var)-cum_%);
MEM-cum: DLT-cum FJ-cum
MEM-cum_%: DLT-cum_% FJ-cum_%;

$(foreach spill, $(SPILLOVER), $(eval $(call SPILL,$(spill))))

$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-ALLSPILL:$(foreach s,$(SPILLOVER),$(var)-$(s)))))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-ALLSPILL_%:$(foreach s,$(SPILLOVER),$(var)-$(s)_%);)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-ALLSPILL:$(foreach s,$(SPILLOVER),$(var)-$(s)))))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-ALLSPILL_%:$(foreach s,$(SPILLOVER),$(var)-$(s)_%);)))
DLT-ALLSPILL: DLT $(foreach var, $(DLT), $(var)-ALLSPILL)
DLT-ALLSPILL_%: DLT_% $(foreach var, $(DLT), $(var)-ALLSPILL_%);
FJ-ALLSPILL: FJ $(foreach var, $(FJ), $(var)-ALLSPILL)
FJ-ALLSPILL_%: FJ_% $(foreach var, $(FJ), $(var)-ALLSPILL_%);
MEM-ALLSPILL: DLT-ALLSPILL FJ-ALLSPILL
MEM-ALLSPILL_%: DLT-ALLSPILL_% FJ-ALLSPILL_%;

$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum-ALLSPILL:$(foreach s,$(SPILLOVER),$(var)-cum-$(s)))))
$(foreach var, $(DLT), $(eval $(call PRINTL,$(var)-cum-ALLSPILL_%:$(foreach s,$(SPILLOVER),$(var)-cum-$(s)_%);)))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum-ALLSPILL:$(foreach s,$(SPILLOVER),$(var)-cum-$(s)))))
$(foreach var, $(FJ), $(eval $(call PRINTL,$(var)-cum-ALLSPILL_%:$(foreach s,$(SPILLOVER),$(var)-cum-$(s)_%);)))
DLT-cum-ALLSPILL: DLT-cum $(foreach var, $(DLT), $(var)-cum-ALLSPILL)
DLT-cum-ALLSPILL_%: DLT-cum_% $(foreach var, $(DLT), $(var)-cum-ALLSPILL_%);
FJ-cum-ALLSPILL: FJ-cum $(foreach var, $(FJ), $(var)-cum-ALLSPILL)
FJ-cum-ALLSPILL_%: FJ-cum_% $(foreach var, $(FJ), $(var)-cum-ALLSPILL_%);
MEM-cum-ALLSPILL: DLT-cum-ALLSPILL FJ-cum-ALLSPILL
MEM-cum-ALLSPILL_%: DLT-cum-ALLSPILL_% FJ-cum-ALLSPILL_%;

KITCHENSINK: MEM-ALLSPILL MEM-cum-ALLSPILL
KITCHENSINK_%: MEM-ALLSPILL_% MEM-cum-ALLSPILL_%;
endef


################################################################################
#
#  Combined toks file creation
#
################################################################################

genmodel/rename_cols.params: | genmodel
	echo '' > $@

%.embd.tokmeasures: %.linetoks $(RT-SCRIPTS)/calcEmbd.py %.nopunc.semprocdecpars
	cat $< | python $(word 2, $^) $(word 3, $^) > $@
    
%.dlt.tokmeasures: %.linetrees $(RT-SCRIPTS)/calcDLT.py
	cat $< | python $(word 2, $^) > $@
  
%.dlt.d.tokmeasures: %.linetrees $(RT-SCRIPTS)/calcDLT.py
	cat $< | python $(word 2, $^) -d > $@

%.pcfg.tokmeasures: %_parsed.output $(LCPARSE-SCRIPTS)/spargeComplexity.py $(RT-SCRIPTS)/calcFJProb.py
	cat $< | python3 $(word 2, $^) | sed 's/\[/!/g;s/\]/!/g;' | python $(word 3, $^) > $@

%.syn.tokmeasures: %.pcfg.tokmeasures $$(word 1, $$(subst ., , %)).gold.dlt.tokmeasures \
$$(word 1, $$(subst ., , %)).gold.embd.tokmeasures $(RT-SCRIPTS)/spilloverMetrics.py
	cat $< | paste -d' ' - <(cut -f 3- -d' ' $(word 2, $^)) <(cut -f 2- -d' ' $(word 3, $^)) | \
	sed 's/  */ /g;1s/F-/fm/g;1s/F+/fp/g;1s/L-/lm/g;1s/L+/lp/g;1s/B-/bm/g;1s/B+/bp/g;1s/Badd/ba/g;1s/Bcdr/bc/g;1s/BNil/bo/g;1s/Bsto/bsto/g;' | \
	python $(word 4, $^) -n3 -p $(DLT) $(FJ) $(EXTRA-SPILL) > $@

%.itemmeasures: $$(basename %).syn.tokmeasures $$(word 1, $$(subst ., , %))$$(subst -,.,$$(suffix $$*)).ngram.itemmeasures \
$(RT-SCRIPTS)/roll_toks.py $(RT-SCRIPTS)/filter_cols.py
	cat $< | python $(word 3, $^) <(cat $(word 2, $^) | sed 's/\[/!/g;s/\]/!/g;') sentid embddepth embddepthS1 embddepthS2 embddepthS3 -e latin-1 | \
	paste -d' ' - <(cut -f 2- -d' ' $(word 2, $^)) | python $(word 4, $^) -d > $@

# Keep old buggy roll operation which messes up sentid
%.o.itemmeasures: $$(basename %).syn.tokmeasures $$(word 1, $$(subst ., , %))$$(subst -,.,$$(suffix $$*)).ngram.itemmeasures \
$(RT-SCRIPTS)/roll_toks.py $(RT-SCRIPTS)/filter_cols.py
	cat <(cat $< | sed 's/\[/!/g;s/\]/!/g;') | python $(word 3, $^) $(word 2, $^) embddepth embddepthS1 embddepthS2 embddepthS3 -e latin-1 | \
	paste -d' ' - <(cut -f 2- -d' ' $(word 2, $^)) | python $(word 4, $^) -d > $@

.PRECIOUS: %.full.evmeasures
%.full.evmeasures: $$(basename %).itemmeasures $$(word 1, $$(subst ., , %))$$(suffix $$*).src.evmeasures \
$(RT-SCRIPTS)/merge_tables.py $(RT-SCRIPTS)/accumulateMetrics.py $(RT-SCRIPTS)/rm_unfix_items.py \
$$(word 1, $$(subst ., , %)).merge_tables.params $$(word 1, $$(subst ., , %)).accumulateMetrics.params \
$$(word 1, $$(subst ., , %)).rm_unfix_items.params
	python $(word 3, $^) $< $(word 2, $^) $(shell cat $(word 6, $^)) | \
	python $(word 4, $^) $(shell cat $(word 7, $^)) | python $(word 5, $^) $(shell cat $(word 8, $^)) > $@

# Just here to preserve old buggy dev/test split
.PRECIOUS: %.o.full.evmeasures
%.o.full.evmeasures: $$(basename %).o.itemmeasures $$(word 1, $$(subst ., , %))$$(suffix $$*).src.o.evmeasures \
$(RT-SCRIPTS)/merge_tables.py $(RT-SCRIPTS)/accumulateMetrics.py $(RT-SCRIPTS)/rm_unfix_items.py \
$$(word 1, $$(subst ., , %)).merge_tables.params $$(word 1, $$(subst ., , %)).accumulateMetrics.params \
$$(word 1, $$(subst ., , %)).rm_unfix_items.params
	python $(word 3, $^) $< $(word 2, $^) $(shell cat $(word 6, $^)) | python $(word 4, $^) $(shell cat $(word 7, $^)) | python $(word 5, $^) $(shell cat $(word 8, $^)) > $@


# Generate and merge parser data from multiple grammars
# Template: mydata.training-GRAMMAR-other-args.stuff.parser1_parser2_parser3.merged.full.evmeasures
.PRECIOUS: %.merged.full.evmeasures
%.merged.full.evmeasures: $(GENERAL-SCRIPTS)/querydataframe.py \
$$(foreach grammar,$$(subst _, ,$$(subst .,,$$(suffix $$*))),genmodel/$$(subst GRAMMAR,$$(grammar),$$(basename $$*)).full.evmeasures)
	@echo 'Newer than target: ' $?
	cp $(word 2, $^) $@
	cat $@ | sed 's/ cumtotsurp / cum$(subst -,,$(word 1, $(subst _, ,$(subst .,,$(suffix $*)))))totsurp /' | \
	sed 's/ totsurp / $(subst -,,$(word 1, $(subst _, ,$(subst .,,$(suffix $*)))))totsurp /' > $@.tmp
	mv $@{.tmp,}
	$(foreach grammar,$(wordlist 2,100,$(subst _, ,$(subst .,,$(suffix $*)))),$(MERGESURP))

################################################################################
#
#  Utility recipes
#
################################################################################

%.evmeasures.tsv: %.evmeasures
	cat $< | sed 's/ /\t/g' > $@

%.tokmeasures.tsv: %.tokmeasures
	cat $< | sed 's/ /\t/g' > $@
