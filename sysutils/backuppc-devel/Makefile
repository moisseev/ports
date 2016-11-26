# Created by: Alexander Moisseev <moiseev@mezonplus.ru>
# $FreeBSD$

PORTNAME=	backuppc
DISTVERSION=	4.0.0alpha3
CATEGORIES=	sysutils
MASTER_SITES=	SF
MASTER_SITE_SUBDIR=	${PORTNAME}/${PORTNAME}-beta/${DISTVERSION}
PKGNAMESUFFIX=	-devel
DISTNAME=	BackupPC-${DISTVERSION}

MAINTAINER=	moiseev@mezonplus.ru
COMMENT=	System for backing PCs and laptops to a server

LICENSE=	GPLv3
LICENSE_FILE=	${WRKSRC}/LICENSE

LATEST_LINK=	${PORTNAME}-devel

RUN_DEPENDS=	p5-File-Listing>=0:${PORTSDIR}/sysutils/p5-File-Listing \
		p5-BackupPC-XS>=0.30:${PORTSDIR}/sysutils/p5-BackupPC-XS

CONFLICTS=	backuppc-[0-9]*

USE_RC_SUBR=	backuppc

USES=		perl5
NO_BUILD=	yes

USERS=		backuppc
GROUPS=		backuppc

DOCSDIR=	${PREFIX}/share/doc/BackupPC
CGIDIR?=	${PREFIX}/www/cgi-bin

SUB_FILES=	pkg-deinstall pkg-message

PLIST_SUB+=	CGIDIR=${CGIDIR:S,^${PREFIX}/,,} \
		DISTVERSION=${DISTVERSION}

PORTDOCS=	ChangeLog LICENSE README

OPTIONS_DEFINE=		ARCHIVE_ZIP DOCS RSYNC_BPC NMBLOOKUP \
			RRDTOOL SMBCLIENT XML_RSS
OPTIONS_DEFAULT=	ARCHIVE_ZIP

ARCHIVE_ZIP_DESC=	Perl module for Zip archive files
RSYNC_BPC_DESC=		Modified rsync that used as part of BackupPC
NMBLOOKUP_DESC=		NetBIOS Name lookup tool
RRDTOOL_DESC=		Round Robin Database Tools
SMBCLIENT_DESC=		Samba client
XML_RSS_DESC=		Perl extension to manage RSS files

.include <bsd.port.options.mk>

.if ${PORT_OPTIONS:MARCHIVE_ZIP}
RUN_DEPENDS+=	p5-Archive-Zip>=0:${PORTSDIR}/archivers/p5-Archive-Zip
.endif
.if ${PORT_OPTIONS:MRSYNC_BPC}
RUN_DEPENDS+=	rsync-bpc>=3.0.9.3:${PORTSDIR}/net/rsync-bpc
.endif
.if ${PORT_OPTIONS:MNMBLOOKUP}
RUN_DEPENDS+=	nmblookup:${PORTSDIR}/net/samba-nmblookup
.endif
.if ${PORT_OPTIONS:MRRDTOOL}
RUN_DEPENDS+=	rrdtool:${PORTSDIR}/databases/rrdtool
.endif
.if ${PORT_OPTIONS:MSMBCLIENT}
RUN_DEPENDS+=	smbclient:${PORTSDIR}/net/samba-smbclient
.endif
.if ${PORT_OPTIONS:MXML_RSS}
RUN_DEPENDS+=	p5-XML-RSS>=0:${PORTSDIR}/textproc/p5-XML-RSS
.endif

post-patch:
	@${REINPLACE_CMD} \
		-e 's|/etc/BackupPC\(/config.pl\)|${ETCDIR}\1|' \
		${WRKSRC}/configure.pl
	@${REINPLACE_CMD} \
		-e 's|STDERR "Please su |STDERR "Please su [-m] |' \
		${WRKSRC}/lib/BackupPC/Lib.pm
	@${REINPLACE_CMD} \
		-e 's| you can use the -s| you can use the -m|' \
		-e 's|option to su to explicitly run|option to su to run|' \
		-e 's|su -s /bin/bash __BACKUPPCUSER__|su -m __BACKUPPCUSER__|' \
		${WRKSRC}/doc/BackupPC.html ${WRKSRC}/doc/BackupPC.pod

do-install:
	if [ -f ${ETCDIR}/config.pl ]; then \
		${MKDIR} ${STAGEDIR}${ETCDIR}; \
		${CP} -p ${ETCDIR}/config.pl ${STAGEDIR}${ETCDIR}/config.pl; \
	fi
	if [ -f ${ETCDIR}/hosts ]; then \
		${MKDIR} ${STAGEDIR}${ETCDIR}; \
		${CP} -p ${ETCDIR}/hosts ${STAGEDIR}${ETCDIR}/hosts; \
	fi

	cd ${WRKSRC} && ${PERL} configure.pl	\
		--batch				\
		--backuppc-user ${USERS}	\
		--config-dir ${ETCDIR}		\
		--cgi-dir ${CGIDIR}		\
		--data-dir /var/db/BackupPC	\
		--dest-dir ${STAGEDIR}		\
		--fhs				\
		--html-dir ${WWWDIR}		\
		--html-dir-url /${PORTNAME}	\
		--install-dir ${PREFIX}		\
		--no-set-perms			\
		--uid-ignore

	pod2man --section=1 --release=${PORTVERSION} \
		--name=BackupPC --center="BackupPC user guide" \
		${WRKSRC}/doc/BackupPC.pod ${WRKSRC}/${PORTNAME}.1
	${INSTALL_MAN} ${WRKSRC}/${PORTNAME}.1 ${STAGEDIR}${MANPREFIX}/man/man1
	cd ${WRKSRC} && ${INSTALL_MAN} ${PORTDOCS} ${STAGEDIR}${DOCSDIR}

	if [ ! -f ${STAGEDIR}${ETCDIR}/config.pl.pre-${DISTVERSION} ]; then \
		> ${STAGEDIR}${ETCDIR}/config.pl.pre-${DISTVERSION}; \
	fi
	${INSTALL} -d ${STAGEDIR}${PREFIX}/libexec/${PORTNAME}
	cd ${STAGEDIR}${ETCDIR} && \
		${CHGRP} wheel config.pl config.pl.pre-${DISTVERSION} && \
		${MV} config.pl hosts config.pl.pre-${DISTVERSION} \
			${STAGEDIR}${PREFIX}/libexec/${PORTNAME}/

.include <bsd.port.mk>
