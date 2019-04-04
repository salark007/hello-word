setwd("D:\\array")
library(Biobase)
library(GEOquery)
library(limma)
series="GSE9476"
gset <- getGEO(series, GSEMatrix =TRUE, AnnotGPL=TRUE, destdir ="data/")
platform="GPL96"
if (length(gset) > 1) idx <- grep(platform, attr(gset, "names")) else idx <- 1
gset <- gset[[idx]]
gr=c("cd34", rep("bm",10), rep("cd34",7), rep("aml",26), rep("pb",10), rep("cd34",10))
ex=exprs(gset)
pdf("result/boxplot.pdf", width = 64)
boxplot(ex)
dev.off()
library(pheatmap)
pdf("result/corheatmap.pdf", width = 15, height = 15)
pheatmap(cor(ex))
dev.off()
pc=prcomp(ex)
pdf("result/pc.pdf")
plot(pc)
plot(pc$x[,1:2])
dev.off()
ex_scale=t(scale(t(ex), scale = F))
pc=prcomp((ex_scale))
pdf("result/pc_scale.pdf")
plot(pc)
plot(pc$x[,1:2])
dev.off()
library(ggplot2)
library(reshape2)
library(plyr)
pdf("result/pca_sample.pdf", width = 15, height = 15)
pcr=data.frame(pc$r[,1:3],Group=gr)
ggplot(pcr, aes(PC1,PC2,color=Group)) + geom_point()
dev.off()
gr=factor(gr)
gset$description <- factor(gr)
design <- model.matrix(~ description + 0, gset)
colnames(design) <- levels(gr)
fit <- lmFit(gset, design)
cont.matrix <- makeContrasts(aml-cd34, levels=design)
fit2 <- contrasts.fit(fit, cont.matrix )
fit2 <- eBayes(fit2, 0.01)
tT <- topTable(fit2, adjust="fdr", sort.by="B", number=Inf)
tT <- subset(tT, select=c("Gene.symbol", "Gene.ID","adj.P.Val","logFC"))
write.table(tT,"result/AML_CD34.txt", row.names=F, sep="\t", quote=F)
