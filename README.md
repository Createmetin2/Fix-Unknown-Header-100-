# Fix-Unknown-Header-100-
I do this topic because I see some people sell while they have taken it for free from me with this topic they will stop selling


SERVERSIDE

0.1


input_udp   
search
Set(1, sizeof(ServerStateChecker_RequestPacket), "ServerStateRequest", false);
change:
Set(1, sizeof(ServerStateChecker_RequestPacket), "ServerStateRequest");


1) open desc.cpp search 

REMOVE:
#include "sequence.h"

REMOVE:
m_iCurrentSequence = 0;

REMOVE: ALL
m_seq_vector.clear();


REMOVE: 
BYTE DESC::GetSequence()
{
	return gc_abSequence[m_iCurrentSequence];
}

void DESC::SetNextSequence()
{
	if (++m_iCurrentSequence == SEQUENCE_MAX_NUM)
		m_iCurrentSequence = 0;
}


REMOVE:
void DESC::push_seq(BYTE hdr, BYTE seq)
{
	if (m_seq_vector.size() >= 20)
	{
		m_seq_vector.erase(m_seq_vector.begin());
	}

	seq_t info = { hdr, seq };
	m_seq_vector.push_back(info);
}








2) OPEN DESC.H

REMOVE:
struct seq_t
{
	BYTE	hdr;
	BYTE	seq;
};
typedef std::vector<seq_t>	seq_vector_t;

REMOVE:
		BYTE			GetSequence();
		void			SetNextSequence();


REMOVE:
int			m_iCurrentSequence;

REMOVE:
	public:
		seq_vector_t	m_seq_vector;
		void			push_seq(BYTE hdr, BYTE seq);










3) OPEN INPUT.CPP

SEARCH:
sys_log(0, "PONG! %u %u", m_pPacketInfo->IsSequence(bHeader), *(BYTE *)(c_pData + iPacketLen - sizeof(BYTE)));

CHANGE:
sys_log(0, "PONG! %u", *(BYTE *)(c_pData + iPacketLen - sizeof(BYTE)));




REMOVE:
		if (m_pPacketInfo->IsSequence(bHeader))
		{
			BYTE bSeq = lpDesc->GetSequence();
			BYTE bSeqReceived = *(BYTE *)(c_pData + iPacketLen - sizeof(BYTE));

			if (bSeq != bSeqReceived)
			{
				sys_err("SEQUENCE %x mismatch 0x%x != 0x%x header %u", get_pointer(lpDesc), bSeq, bSeqReceived, bHeader);

				LPCHARACTER	ch = lpDesc->GetCharacter();

				char buf[1024];
				int	offset, len;

				offset = snprintf(buf, sizeof(buf), "SEQUENCE_LOG [%s]-------------\n", ch ? ch->GetName() : "UNKNOWN");

				if (offset < 0 || offset >= (int) sizeof(buf))
					offset = sizeof(buf) - 1;

				for (size_t i = 0; i < lpDesc->m_seq_vector.size(); ++i)
				{
					len = snprintf(buf + offset, sizeof(buf) - offset, "\t[%03d : 0x%x]\n",
						lpDesc->m_seq_vector[i].hdr,
						lpDesc->m_seq_vector[i].seq);

					if (len < 0 || len >= (int) sizeof(buf) - offset)
						offset += (sizeof(buf) - offset) - 1;
					else
						offset += len;
				}

				snprintf(buf + offset, sizeof(buf) - offset, "\t[%03d : 0x%x]\n", bHeader, bSeq);
				sys_err("%s", buf);

				lpDesc->SetPhase(PHASE_CLOSE);
				return true;
			}
			else
			{
				lpDesc->push_seq(bHeader, bSeq);
				lpDesc->SetNextSequence();
				//sys_err("SEQUENCE %x match %u next %u header %u", lpDesc, bSeq, lpDesc->GetSequence(), bHeader);
			}
		}




REMOVE:
pkPacketInfo->SetSequence(HEADER_CG_PONG, false);


















4)OPEN PACKET_INFO.CPP

SEARCH:
void CPacketInfo::Set(int header, int iSize, const char * c_pszName, bool bSeq)

CHANGE:
void CPacketInfo::Set(int header, int iSize, const char * c_pszName)


REMOVE:
	element->bSequencePacket = bSeq;

	if (element->bSequencePacket)
		element->iSize += sizeof(BYTE);


REMOVE:
bool CPacketInfo::IsSequence(int header)
{
	TPacketElement * pkElement = GetElement(header);
	return pkElement ? pkElement->bSequencePacket : false;
}

void CPacketInfo::SetSequence(int header, bool bSeq)
{
	TPacketElement * pkElem = GetElement(header);

	if (pkElem)
	{
		if (bSeq)
		{
			if (!pkElem->bSequencePacket)
				pkElem->iSize++;
		}
		else
		{
			if (pkElem->bSequencePacket)
				pkElem->iSize--;
		}

		pkElem->bSequencePacket = bSeq;
	}
}


SEARCH :
void CPacketInfo::Log(const char * c_pszFileName)

CHANGE WITH :

void CPacketInfo::Log(const char * c_pszFileName)
{
#ifndef NO_PACKET_INFO_LOG
	FILE * fp;

	fp = fopen(c_pszFileName, "w");

	if (!fp)
		return;

	std::map<int, TPacketElement *>::iterator it = m_pPacketMap.begin();

	fprintf(fp, "Name             Called     Load       Ratio\n");

	while (it != m_pPacketMap.end())
	{
		TPacketElement * p = it->second;
		++it;

		fprintf(fp, "%-16s %-10d %-10u %.2f\n",
				p->stName.c_str(),
				p->iCalled,
				p->dwLoad,
				p->iCalled != 0 ? (float) p->dwLoad / p->iCalled : 0.0f);
	}

	fclose(fp);
#endif
}



REPLASE:

, false");

to
");

you know you see in packet --->  , false  remove 

test:
	Set(HEADER_CG_TEXT, sizeof(TPacketCGText), "Text");
	

















5) OPEN PACKET_INFO.H


REMOVE:
	bool	bSequencePacket;


SEARCH:
void Set(int header, int size, const char * c_pszName, bool bSeq = false);

CHANGE:
void Set(int header, int size, const char * c_pszName);


REMOVE:
		bool IsSequence(int header);
		void SetSequence(int header, bool bSeq);








6) OPEN SEQUENCE.CPP

REMOVE ONLY 

const BYTE gc_abSequence[SEQUENCE_MAX_NUM] =
{
..................BLA BLA
...............BLA BLA 
............. BLA BLA
.........BLA BLA BLA BLA
};





7) OPEN SEQUENCE.H
REMOVE ALL








CLIENTSIDE 





REMOVE:
m_iSequence = 0;



REMOVE ALL:
#define SEQUENCE_TABLE_SIZE 32768


REMOVE:
void CNetworkStream::SetPacketSequenceMode(bool isOn)
{
	m_bUseSequence = isOn;
}



SEARCH:
bool CNetworkStream::SendSequence()

CHANGE WITH :
bool CNetworkStream::SendSequence()
{
	return true;
}

SEARCH:
static BYTE s_bSequenceTable[SEQUENCE_TABLE_SIZE] =

REMOVE:
static BYTE s_bSequenceTable[SEQUENCE_TABLE_SIZE] =
{
..................BLA BLA
...............BLA BLA 
............. BLA BLA
.........BLA BLA BLA BLA
};



REMOVE:
	m_iSequence = 0;
	m_bUseSequence = false;
	m_kVec_bSequenceTable.resize(SEQUENCE_TABLE_SIZE);
	memcpy(&m_kVec_bSequenceTable[0], s_bSequenceTable, sizeof(BYTE) * SEQUENCE_TABLE_SIZE);



OPEN NETSTREAM.H

REMOVE:
void SetPacketSequenceMode(bool isOn);

REMOVE:
		DWORD					m_iSequence;
		bool					m_bUseSequence;
		std::vector<BYTE>		m_kVec_bSequenceTable;



OPEN PYTHONNETWORKSTREAMMODULE.CPP


GO IN  PyObject* netSetPacketSequenceMode(PyObject* poSelf, PyObject* poArgs)

REMOVE THIS
	CPythonNetworkStream& rns = CPythonNetworkStream::Instance();
	CAccountConnector & rkAccountConnector = CAccountConnector::Instance();
	rns.SetPacketSequenceMode(true);
	rkAccountConnector.SetPacketSequenceMode(true);









1) EDITOR


#ifdef IMPROVED_PACKET_ENCRYPTION

NO!!!!!!!!!  NOOOO
ifndef IMPROVED_PACKET_ENCRYPTION 
NO REMOVE REMOVE ONLY THIS #ifdef IMPROVED_PACKET_ENCRYPTION

REMOVE ALL CODE #ifdef IMPROVED_PACKET_ENCRYPTION

#ifdef = REMOVE
#ifndef = NO REMOVE NOOOO
---------------------------------------------------------------------------------------------------------------









---------------------------------------------------------------------------------------------------------------
1.0 open desc.cpp
search:
void DESC::Packet(const void * c_pvData, int iSize)
{
	assert(iSize > 0);

change:
void DESC::Packet(const void * c_pvData, int iSize)
{
	assert(NULL != c_pvData);
	assert(iSize > 0);
---------------------------------------------------------------------------------------------------------------







---------------------------------------------------------------------------------------------------------------
1.1) open desc.cpp search 
		else if (!m_bEncrypted)
		
change:
	else if (!m_bEncrypted)
	{
		int iBytesProceed = 0;

		while (!m_pInputProcessor->Process(this, buffer_read_peek(m_lpInputBuffer), buffer_size(m_lpInputBuffer), iBytesProceed))
		{
			buffer_read_proceed(m_lpInputBuffer, iBytesProceed);
			iBytesProceed = 0;
		}

		buffer_read_proceed(m_lpInputBuffer, iBytesProceed);
	}
	else
	{
		int iSizeBuffer = buffer_size(m_lpInputBuffer);

		if (iSizeBuffer & 7)
			iSizeBuffer -= iSizeBuffer & 7;

		if (iSizeBuffer > 0)
		{
			TEMP_BUFFER	tempbuf(8192 + 4);
			LPBUFFER lpBufferDecrypt = tempbuf.getptr();
			buffer_adjust_size(lpBufferDecrypt, iSizeBuffer);

			long iSizeAfter = TEA_Decrypt(reinterpret_cast<DWORD *>(buffer_write_peek(lpBufferDecrypt)),
					reinterpret_cast<const DWORD *>( buffer_read_peek(m_lpInputBuffer)),
					GetDecryptionKey(),
					iSizeBuffer);

			buffer_write_proceed(lpBufferDecrypt, iSizeAfter);

			int iBytesProceed = 0;

			while (!m_pInputProcessor->Process(this, buffer_read_peek(lpBufferDecrypt), buffer_size(lpBufferDecrypt), iBytesProceed))
			{
				if (iBytesProceed > iSizeBuffer)
				{
					buffer_read_proceed(m_lpInputBuffer, iSizeBuffer);
					iSizeBuffer = 0;
					iBytesProceed = 0;
					break;
				}

				buffer_read_proceed(m_lpInputBuffer, iBytesProceed);
				iSizeBuffer -= iBytesProceed;

				buffer_read_proceed(lpBufferDecrypt, iBytesProceed);
				iBytesProceed = 0;
			}

			buffer_read_proceed(m_lpInputBuffer, iBytesProceed);
		}
	}

	return (bytes_read);
}
---------------------------------------------------------------------------------------------------------------







---------------------------------------------------------------------------------------------------------------
1.2) search in desc.cpp 
		if (!m_bEncrypted)
		

change: 

		if (!m_bEncrypted)
		{
			if (!packet_encode(m_lpOutputBuffer, c_pvData, iSize))
			{
				m_iPhase = PHASE_CLOSE;
			}
		}
		else
		{
			if (buffer_has_space(m_lpOutputBuffer) < iSize + 8)
			{
				buffer_adjust_size(m_lpOutputBuffer, iSize);
				if (buffer_has_space(m_lpOutputBuffer) < iSize + 8)
				{
					sys_err(
						"desc buffer mem_size overflow : ",
						"	memsize(%u) ",
						"	write_pos(%u)",
						"	iSize(%d)",
						m_lpOutputBuffer->mem_size,
						m_lpOutputBuffer->write_point_pos,
						iSize);
					m_iPhase = PHASE_CLOSE;
				}
			}
			else
			{
				DWORD * pdwWritePoint = static_cast<DWORD *>(buffer_write_peek(m_lpOutputBuffer));

				if (packet_encode(m_lpOutputBuffer, c_pvData, iSize))
				{
					long iSize2 = TEA_Encrypt(pdwWritePoint, pdwWritePoint, GetEncryptionKey(), iSize);

					if (iSize2 > iSize)
						buffer_write_proceed(m_lpOutputBuffer, iSize2 - iSize);
				}
			}
		}

		SAFE_BUFFER_DELETE(m_lpBufferedOutputBuffer);
	}

	if (m_iPhase != PHASE_CLOSE)
		fdwatch_add_fd(m_lpFdw, m_sock, this, FDW_WRITE, true);
}
---------------------------------------------------------------------------------------------------------------







---------------------------------------------------------------------------------------------------------------
1.4 open protocol.h search
//buffer_adjust_size(pbuf, length);

change :
buffer_adjust_size(pbuf, length);
---------------------------------------------------------------------------------------------------------------



















ENABLE PROTECTION IN PACKET


1) OPEN SERVICE.CPP
ADD
#define ENABLE_ANTI_PACKET_FLOOD


2) OPEN CHAR.CPP 
SEARCH
	m_fAttMul = 1.0f;
	m_fDamMul = 1.0f;

ADD
#ifdef ENABLE_ANTI_PACKET_FLOOD
	m_dwPacketAntiFloodCount = 0;
	m_dwPacketAntiFloodPulse = 0;
#endif

3) OPEN CHAR.H
SEARCH
	protected:
		CARDS_INFO	character_cards;
		S_CARD	randomized_cards[24];

ADD
#ifdef ENABLE_ANTI_PACKET_FLOOD
	private:
		int				m_dwPacketAntiFloodPulse;
		DWORD			m_dwPacketAntiFloodCount;
	public:
		int				GetPacketAntiFloodPulse(){return m_dwPacketAntiFloodPulse;}
		DWORD			GetPacketAntiFloodCount(){return m_dwPacketAntiFloodCount;}
		DWORD			IncreasePacketAntiFloodCount(){return ++m_dwPacketAntiFloodCount;}
		void			SetPacketAntiFloodPulse(int dwPulse){m_dwPacketAntiFloodPulse=dwPulse;}
		void			SetPacketAntiFloodCount(DWORD dwCount){m_dwPacketAntiFloodCount=dwCount;}
#endif


4) OPEN INPUT.CPP
SEARCH
			if (iExtraPacketSize < 0)
			{

CHANGE
			if (iExtraPacketSize < 0)
			{
#ifdef ENABLE_ANTI_PACKET_FLOOD
				sys_err("Failed to analyze header(%u) host(%s)", bHeader, inet_ntoa(lpDesc->GetAddr().sin_addr));
				lpDesc->SetPhase(PHASE_CLOSE);
#endif

5) OPEN INPUT_LOGIN.CPP  IN   int CInputLogin::Analyze(LPDESC d, BYTE bHeader, const char * c_pData)


CHANGE ALL default:


		default:
			sys_err("login phase does not handle this packe2t! header %d", bHeader);
#ifdef ENABLE_ANTI_PACKET_FLOOD
			return -1;
#else
			return (0);
#endif
	}

	return (iExtraLen);
}


6) OPEN INPUT_MAIN.CPP
SEARCH
	if (!(ch = d->GetCharacter()))

CHANGE

	if (!(ch = d->GetCharacter()))
	{
		sys_err("no character on desc");
		d->SetPhase(PHASE_CLOSE);
#ifdef ENABLE_ANTI_PACKET_FLOOD
		return -1;
#else
		return (0);
#endif
	}

#ifdef ENABLE_ANTI_PACKET_FLOOD
	if (bHeader != HEADER_CG_ITEM_USE)
	{
		if (thecore_pulse() > ch->GetPacketAntiFloodPulse() + PASSES_PER_SEC(1))
		{
			ch->SetPacketAntiFloodCount(0);
			ch->SetPacketAntiFloodPulse(thecore_pulse());
		}
		
		if (ch->IncreasePacketAntiFloodCount() >= 250)
		{
			sys_err("<Flood packet> name(%s) header(%u) host(%s)", ch->GetName(), bHeader, inet_ntoa(d->GetAddr().sin_addr));
			ch->GetDesc()->DelayedDisconnect(0);
			return false;
		}
	}
#endif


SEARCH 
if (!(ch = d->GetCharacter()))

CHANGE

	if (!(ch = d->GetCharacter()))
	{
		sys_err("no character on desc");
#ifdef ENABLE_ANTI_PACKET_FLOOD
		return -1;
#else
		return 0;
#endif
	}



IN int CInputDead::Analyze(LPDESC d, BYTE bHeader, const char * c_pData)


		default:  CHANGE ALL

		default:  CHANGE 
#ifdef ENABLE_ANTI_PACKET_FLOOD
			return -1;
#else
			return (0);
#endif
	}

	return (iExtraLen);
}





CLIENT:
1) REMOVE ALL SEQUENCE IN CLIENT  SendSEquence SEARCH AND REMOVE


General delete the sequence + sendsequence + disable packet





---------KEY CLIENT FOR PROTECT SERVER CLIENT----------------
PLEASE CREATE KEY IN LIB FOR PROTECT YOUR SERVER :) KEY YOU KNOW WHAT ADD KEY CREATE WITH YOUR METHOD KEY
--------------------------------------------------------------------------------------------------------------


---------KEY CLIENT FOR PROTECT SERVER CLIENT----------------
PLEASE CREATE KEY IN LIB FOR PROTECT YOUR SERVER :) KEY YOU KNOW WHAT ADD KEY CREATE WITH YOUR METHOD KEY
--------------------------------------------------------------------------------------------------------------


---------KEY CLIENT FOR PROTECT SERVER CLIENT----------------
PLEASE CREATE KEY IN LIB FOR PROTECT YOUR SERVER :) KEY YOU KNOW WHAT ADD KEY CREATE WITH YOUR METHOD KEY
--------------------------------------------------------------------------------------------------------------







I can help you in this mess without money






















