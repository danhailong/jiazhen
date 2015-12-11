12 /09 日 
// Copydlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Copydlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CCopydlg dialog

extern CMyApp theApp;
CCopydlg::CCopydlg(CWnd* pParent /*=NULL*/)
	: CDialog(CCopydlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CCopydlg)
	m_Name = _T("");
	//}}AFX_DATA_INIT
}


void CCopydlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CCopydlg)
	DDX_Control(pDX, IDC_EDIT1, m_Edit);
	DDX_Text(pDX, IDC_EDIT2, m_Name);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CCopydlg, CDialog)
	//{{AFX_MSG_MAP(CCopydlg)
	ON_BN_CLICKED(IDC_BUTTON_checkml, OnBUTTONcheckml)
	ON_BN_CLICKED(IDC_BUTTON_OK, OnButtonOk)
	ON_BN_CLICKED(IDC_BUTTON_CANCEL, OnButtonCancel)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CCopydlg message handlers

void CCopydlg::OnBUTTONcheckml() 
{
	CString ReturnPach;
	TCHAR szPath[_MAX_PATH];
	BROWSEINFO bi;
	bi.hwndOwner=NULL;
	bi.pidlRoot=NULL;
	bi.lpszTitle=_T("请选择备份文件夹");
	bi.pszDisplayName=szPath;
	bi.ulFlags=BIF_RETURNONLYFSDIRS;
	bi.lpfn=NULL;
	bi.lParam=NULL;
	LPITEMIDLIST pItemIDList=SHBrowseForFolder(&bi);
	if(pItemIDList)
	{
		if(SHGetPathFromIDList(pItemIDList,szPath))
			ReturnPach=szPath;	
	}
	else
		ReturnPach="";
	m_Edit.SetWindowText(ReturnPach);	
}

void CCopydlg::OnButtonOk() 
{
	UpdateData();
	CString str,strpath;
	m_Edit.GetWindowText(str);
	strpath = str+"\\"+m_Name+".mdb";
	char buf[256];
	::GetCurrentDirectory(256,buf);
	strcat(buf,"\\canyin.mdb");
	CopyFile(buf,strpath,false);//复制文件
	MessageBox("备份完成！","系统提示",MB_OK|MB_ICONEXCLAMATION);
	CDialog::OnOK();
}


void CCopydlg::OnButtonCancel() 
{
	CDialog::OnCancel();	
}





// CPdlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "CPdlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CCPdlg dialog

extern CMyApp theApp;
CCPdlg::CCPdlg(CWnd* pParent /*=NULL*/)
	: CDialog(CCPdlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CCPdlg)
	m_Name = _T("");
	m_Price = _T("");
	//}}AFX_DATA_INIT
}


void CCPdlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CCPdlg)
	DDX_Control(pDX, IDC_LIST1, m_CpList);
	DDX_Text(pDX, IDC_EDIT_name, m_Name);
	DDX_Text(pDX, IDC_EDIT_price, m_Price);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CCPdlg, CDialog)
	//{{AFX_MSG_MAP(CCPdlg)
	ON_BN_CLICKED(IDC_BUTTON1, OnButtonadd)
	ON_BN_CLICKED(IDC_BUTTON2, OnButton2)
	ON_NOTIFY(NM_DBLCLK, IDC_LIST1, OnDblclkList1)
	ON_BN_CLICKED(IDC_BUTTON_change, OnBUTTONchange)
	ON_BN_CLICKED(IDC_BUTTON_del, OnBUTTONdel)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CCPdlg message handlers

BOOL CCPdlg::OnInitDialog()
{
	CDialog::OnInitDialog();
	m_CpList.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_CpList.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_CpList.InsertColumn(1,"菜价(元)",LVCFMT_LEFT,100,1);
	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from caishiinfo"),NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		m_CpList.InsertItem(0,"");
		m_CpList.SetItemText(0,0,(char*)(_bstr_t)m_pRs->GetCollect("菜名"));
		m_CpList.SetItemText(0,1,(char*)(_bstr_t)m_pRs->GetCollect("菜价"));
		m_pRs->MoveNext();
	}
	return true;
}

void CCPdlg::OnButtonadd() 
{
	UpdateData();
	if(m_Name.IsEmpty())
	{
		AfxMessageBox("菜名不能为空");
		return;
	}
	if(m_Price.IsEmpty())
	{
		AfxMessageBox("菜价不能为空");
		return;
	}

	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from caishiinfo where 菜名='"+m_Name+"'"),NULL,adCmdText);
	if(!m_pRs->adoEOF)
	{
		AfxMessageBox("已经有这个菜了");
		return;
	}
	else
	{
		CString sql="insert into caishiinfo (菜名,菜价) values ('"+m_Name+"',"+m_Price+")";
		theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
		m_CpList.DeleteAllItems();
		m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from caishiinfo"),NULL,adCmdText);
		while(!m_pRs->adoEOF)
		{		
			m_CpList.InsertItem(0,"");
			m_CpList.SetItemText(0,0,(char*)(_bstr_t)m_pRs->GetCollect("菜名"));
			m_CpList.SetItemText(0,1,(char*)(_bstr_t)m_pRs->GetCollect("菜价"));
			m_pRs->MoveNext();			
		}
		m_Name="";
		m_Price="";
		UpdateData(false);			
	}
}

void CCPdlg::OnButton2() 
{
	CDialog::OnCancel();	
}

void CCPdlg::OnDblclkList1(NMHDR* pNMHDR, LRESULT* pResult) 
{
	int i=m_CpList.GetSelectionMark();
	CString str=m_CpList.GetItemText(i,0);
	CString str1=m_CpList.GetItemText(i,1);
	m_Name=str;
	m_Price=str1;
	UpdateData(false);
	*pResult = 0;
}

void CCPdlg::OnBUTTONchange() 
{
	UpdateData();
	if(m_Name.IsEmpty())
	{
		AfxMessageBox("菜名不能为空");
		return;
	}
	if(m_Price.IsEmpty())
	{
		AfxMessageBox("菜价不能为空");
		return;
	}
	CString sql="update caishiinfo set 菜价="+m_Price+" where 菜名='"+m_Name+"'";
	theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	m_CpList.DeleteAllItems();
	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from caishiinfo"),NULL,adCmdText);
	while(!m_pRs->adoEOF)
	{		
		m_CpList.InsertItem(0,"");
		m_CpList.SetItemText(0,0,(char*)(_bstr_t)m_pRs->GetCollect("菜名"));
		m_CpList.SetItemText(0,1,(char*)(_bstr_t)m_pRs->GetCollect("菜价"));
		m_pRs->MoveNext();	
	}
	m_Name="";
	m_Price="";
	UpdateData(false);	
}

void CCPdlg::OnBUTTONdel() 
{
	UpdateData();
	if(m_Name.IsEmpty())
	{
		AfxMessageBox("菜名不能为空");
		return;
	}
	if(m_Price.IsEmpty())
	{
		AfxMessageBox("菜价不能为空");
		return;
	}
	CString sql="delete from caishiinfo where 菜价="+m_Price+" and 菜名='"+m_Name+"'";
	theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	m_CpList.DeleteAllItems();
	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from caishiinfo"),NULL,adCmdText);
	while(!m_pRs->adoEOF)
	{		
		m_CpList.InsertItem(0,"");
		m_CpList.SetItemText(0,0,(char*)(_bstr_t)m_pRs->GetCollect("菜名"));
		m_CpList.SetItemText(0,1,(char*)(_bstr_t)m_pRs->GetCollect("菜价"));
		m_pRs->MoveNext();			
		m_Name="";
		m_Price="";
		UpdateData(false);
	}
	
}






// Diancaidlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Diancaidlg.h"
#include "kaitaidlg.h"
#include "SLdlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CDiancaidlg dialog
extern CMyApp theApp;


CDiancaidlg::CDiancaidlg(CWnd* pParent /*=NULL*/)
	: CDialog(CDiancaidlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CDiancaidlg)
	m_ZhuoHao = _T("");
	//}}AFX_DATA_INIT
}


void CDiancaidlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CDiancaidlg)
	DDX_Control(pDX, IDC_LIST3, m_CaidanCheck);
	DDX_Control(pDX, IDC_LIST2, m_CaidanList);
	DDX_Text(pDX, IDC_EDIT_zhuohao, m_ZhuoHao);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CDiancaidlg, CDialog)
	//{{AFX_MSG_MAP(CDiancaidlg)
	ON_BN_CLICKED(IDC_BUTTON1, OnButtonadd)
	ON_BN_CLICKED(IDC_BUTTON_sub, OnBUTTONsub)
	ON_BN_CLICKED(IDC_BUTTON_OK, OnButtonOk)
	ON_BN_CLICKED(IDC_BUTTON_CANCEL, OnButtonCancel)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CDiancaidlg message handlers




BOOL CDiancaidlg::OnInitDialog()
{
	CDialog::OnInitDialog();
	SetIcon(LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_ICON_diancai)),TRUE);
	CString Sql="select * from caishiinfo";
	m_CaidanList.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_CaidanList.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_CaidanList.InsertColumn(1,"菜价(元)",LVCFMT_LEFT,100,1);
	m_pRs=theApp.m_pCon->Execute((_bstr_t)Sql,NULL,adCmdText);
	while(!m_pRs->adoEOF)
	{
		CString TheValue,TheValue1;
		TheValue=(char*)(_bstr_t)m_pRs->GetCollect("菜名");
		TheValue1=(char*)(_bstr_t)m_pRs->GetCollect("菜价");
		m_CaidanList.InsertItem(0,"");
		m_CaidanList.SetItemText(0,0,TheValue);
		m_CaidanList.SetItemText(0,1,TheValue1);
		m_pRs->MoveNext();
	}
	m_CaidanCheck.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_CaidanCheck.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_CaidanCheck.InsertColumn(1,"数量(盘)",LVCFMT_LEFT,100,1);	
	return TRUE;
}

void CDiancaidlg::OnButtonadd() 
{
	CSLdlg Sldlg;
	if(Sldlg.DoModal()==IDOK)
	{		
		int i = m_CaidanList.GetSelectionMark();
		CString str = m_CaidanList.GetItemText(i,0);
		m_CaidanCheck.InsertItem(0,"");
		m_CaidanCheck.SetItemText(0,0,str);
		m_CaidanCheck.SetItemText(0,1,Sldlg.m_ShuLiang);	
	}
}

void CDiancaidlg::OnBUTTONsub() 
{
	m_CaidanCheck.DeleteItem(m_CaidanCheck.GetSelectionMark());
}

void CDiancaidlg::OnButtonOk() 
{
	UpdateData();
	CString Sql;
	int i = m_CaidanCheck.GetItemCount();
	if(i==0)
	{
		AfxMessageBox("请点菜");
		return;
	}
	Sql="update TableUSE set TableUSEID=1 where 桌号="+m_ZhuoHao+" ";
	theApp.m_pCon->Execute((_bstr_t)Sql,NULL,adCmdText);
	CString Sql1,Str,Str1,Value,TotleValue;
	double Totle=0;
	for(int n=0;n<i;n++)
	{
		Str=m_CaidanCheck.GetItemText(n,0);
		Str1=m_CaidanCheck.GetItemText(n,1);
		Sql1="select * from caishiinfo where 菜名='"+Str+"'";
		m_pRs=theApp.m_pCon->Execute((_bstr_t)Sql1,NULL,adCmdText);
		Value=(char*)(_bstr_t)m_pRs->GetCollect("菜价");
		Totle=atof(Value)*atof(Str1);
		TotleValue=(char*)(_bstr_t)Totle;
		Sql1="insert into paybill(桌号,菜名,数量,消费) values("+m_ZhuoHao+",'"+Str+"',"+Str1+","+TotleValue+")";
		theApp.m_pCon->Execute((_bstr_t)Sql1,NULL,adCmdText);
	}
	AfxMessageBox("点菜成功");
	CDialog::OnOK();	
}



void CDiancaidlg::OnButtonCancel() 
{
	CDialog::OnCancel();
	
}


void CDiancaidlg::OnOK()
{
	OnButtonOk();
}



// 12/10 日 
// Jiacaidlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Jiacaidlg.h"
#include "Sldlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CJiacaidlg dialog

extern CMyApp theApp;
CJiacaidlg::CJiacaidlg(CWnd* pParent /*=NULL*/)
	: CDialog(CJiacaidlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CJiacaidlg)
		// NOTE: the ClassWizard will add member initialization here
	//}}AFX_DATA_INIT
}


void CJiacaidlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CJiacaidlg)
	DDX_Control(pDX, IDC_COMBO1, m_ZhuohaoCombo);
	DDX_Control(pDX, IDC_LIST3, m_CaidanCheck);
	DDX_Control(pDX, IDC_LIST2, m_CaidanList);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CJiacaidlg, CDialog)
	//{{AFX_MSG_MAP(CJiacaidlg)
	ON_CBN_SELCHANGE(IDC_COMBO1, OnSelchangeCombo1)
	ON_BN_CLICKED(IDC_BUTTON1, OnButtonadd)
	ON_BN_CLICKED(IDC_BUTTON2, OnButtonsub)
	ON_BN_CLICKED(IDC_BUTTON3, OnButtonOK)
	ON_BN_CLICKED(IDC_BUTTON4, OnButton4)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CJiacaidlg message handlers

BOOL CJiacaidlg::OnInitDialog() 
{
	CDialog::OnInitDialog();
	SetIcon(LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_ICON_diancai)),TRUE);
	CString Sql="select * from caishiinfo";
	m_CaidanList.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_CaidanList.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_CaidanList.InsertColumn(1,"菜价(元)",LVCFMT_LEFT,100,1);
	m_pRs=theApp.m_pCon->Execute((_bstr_t)Sql,NULL,adCmdText);
	while(!m_pRs->adoEOF)
	{
		CString TheValue,TheValue1;
		TheValue=(char*)(_bstr_t)m_pRs->GetCollect("菜名");
		TheValue1=(char*)(_bstr_t)m_pRs->GetCollect("菜价");
		m_CaidanList.InsertItem(0,"");
		m_CaidanList.SetItemText(0,0,TheValue);
		m_CaidanList.SetItemText(0,1,TheValue1);
		m_pRs->MoveNext();
	}
	m_CaidanCheck.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_CaidanCheck.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_CaidanCheck.InsertColumn(1,"数量(盘)",LVCFMT_LEFT,100,1);

	Sql="select distinct 桌号 from paybill";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)Sql,NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		CString zhuohao=(char*)(_bstr_t)m_pRs->GetCollect("桌号");
		m_ZhuohaoCombo.AddString(zhuohao);
		m_pRs->MoveNext();
	}
	
	
	return TRUE;  // return TRUE unless you set the focus to a control
	              // EXCEPTION: OCX Property Pages should return FALSE
}

void CJiacaidlg::OnSelchangeCombo1() 
{
	CString str;
	m_ZhuohaoCombo.GetLBText(m_ZhuohaoCombo.GetCurSel(),str);
	CString sql="select * from paybill where 桌号="+str+"";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	m_CaidanCheck.DeleteAllItems();
	while(!m_pRs->adoEOF)
	{
		CString valuename=(char*)(_bstr_t)m_pRs->GetCollect("菜名");
		CString valuenum=(char*)(_bstr_t)m_pRs->GetCollect("数量");
		m_CaidanCheck.InsertItem(0,"");
		m_CaidanCheck.SetItemText(0,0,valuename);
		m_CaidanCheck.SetItemText(0,1,valuenum);
		m_pRs->MoveNext();
	}

}

void CJiacaidlg::OnButtonadd() 
{
	CSLdlg Sldlg;
	if(Sldlg.DoModal()==IDOK)
	{		
		int i = m_CaidanList.GetSelectionMark();
		CString str = m_CaidanList.GetItemText(i,0);
		m_CaidanCheck.InsertItem(0,"");
		m_CaidanCheck.SetItemText(0,0,str);
		m_CaidanCheck.SetItemText(0,1,Sldlg.m_ShuLiang);	
	}	
}

void CJiacaidlg::OnButtonsub() 
{
	m_CaidanCheck.DeleteItem(m_CaidanCheck.GetSelectionMark());	
}

void CJiacaidlg::OnButtonOK() 
{
	UpdateData();
	CString Sql;
	CString zhuohao;
	if(m_ZhuohaoCombo.GetCurSel()==-1)
	{
		AfxMessageBox("请选择要加菜的桌号");
		return;
	}
	m_ZhuohaoCombo.GetLBText(m_ZhuohaoCombo.GetCurSel(),zhuohao);
	int i = m_CaidanCheck.GetItemCount();
	if(i==0)
	{
		AfxMessageBox("请点菜");
		return;
	}
	CString Sql1,Str,Str1,Value,TotleValue;
	Sql1="delete from paybill where 桌号="+zhuohao+"";
	theApp.m_pCon->Execute((_bstr_t)Sql1,NULL,adCmdText);
	double Totle=0;
	for(int n=0;n<i;n++)
	{
	Str=m_CaidanCheck.GetItemText(n,0);
	Str1=m_CaidanCheck.GetItemText(n,1);
	Sql1="select * from caishiinfo where 菜名='"+Str+"'";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)Sql1,NULL,adCmdText);
	Value=(char*)(_bstr_t)m_pRs->GetCollect("菜价");
	Totle=atof(Value)*atof(Str1);
	TotleValue=(char*)(_bstr_t)Totle;
	Sql1="insert into paybill(桌号,菜名,数量,消费) values("+zhuohao+",'"+Str+"',"+Str1+","+TotleValue+")";
	theApp.m_pCon->Execute((_bstr_t)Sql1,NULL,adCmdText);
	}
	AfxMessageBox("操作成功");
	CDialog::OnOK();		
		
}

void CJiacaidlg::OnButton4() 
{
	CDialog::OnCancel();	
}





// Jiezhangdlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Jiezhangdlg.h"


#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CJiezhangdlg dialog

extern CMyApp theApp;
CJiezhangdlg::CJiezhangdlg(CWnd* pParent /*=NULL*/)
	: CDialog(CJiezhangdlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CJiezhangdlg)
	//}}AFX_DATA_INIT
}


void CJiezhangdlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CJiezhangdlg)
	DDX_Control(pDX, IDC_mingxi, m_MingXi);
	DDX_Control(pDX, IDC_zhaoling, m_ZhaoLing);
	DDX_Control(pDX, IDC_yingshou, m_YingShou);
	DDX_Control(pDX, IDC_shishou, m_ShiShou);
	DDX_Control(pDX, IDC_COMBO1, m_Combo);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CJiezhangdlg, CDialog)
	//{{AFX_MSG_MAP(CJiezhangdlg)
	ON_BN_CLICKED(IDC_BUTTON_OK2, OnButtonOk2)
	ON_BN_CLICKED(IDC_BUTTON_BYE, OnButtonBye)
	ON_EN_CHANGE(IDC_EDIT_shishou, OnChangeEDITshishou)
	ON_CBN_SELCHANGE(IDC_COMBO1, OnSelchangeCombo1)
	ON_CBN_EDITCHANGE(IDC_COMBO1, OnEditchangeCombo1)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CJiezhangdlg message handlers

BOOL CJiezhangdlg::OnInitDialog()
{
	CDialog::OnInitDialog();
	SetIcon(LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_ICON_pay)),TRUE);
	CString TheValue;
	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from TableUSE where TableUSEID=1"),NULL,adCmdText);
	if(m_pRs->GetRecordCount()==0)
		return true;
	if(m_pRs->GetRecordCount()==1)
	{
		TheValue=(char*)(_bstr_t)m_pRs->GetCollect("桌号");	
		m_Combo.AddString(TheValue);
		return true;
	}
	
	while(!m_pRs->adoEOF)
	{
		TheValue=(char*)(_bstr_t)m_pRs->GetCollect("桌号");		
		m_Combo.AddString(TheValue);
		m_pRs->MoveNext();
	}
	m_pRs=NULL;
	m_MingXi.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_MingXi.InsertColumn(0,"菜名",LVCFMT_LEFT,100,0);
	m_MingXi.InsertColumn(1,"数量",LVCFMT_LEFT,100,1);
	m_MingXi.InsertColumn(2,"消费(元)",LVCFMT_LEFT,120,1);
	res = FALSE;
	return true;


}


void CJiezhangdlg::OnButtonOk2() 
{
	UpdateData();
	CString str,str1,str2,str3;
	CString TheValue;
	CString ShiShou,YingShou;
	m_Combo.GetWindowText(str1);
	if(str1.GetLength()<4||str1.GetLength()>4)
	{
		AfxMessageBox("输入错误");
		return;
	}
	CString bjsql="select * from TableUSE where 桌号="+str1+"";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)bjsql,NULL,adCmdText);
	if(m_pRs->adoEOF)
	{
		AfxMessageBox("没有这张餐台");
		return;
	}
	CString bjstr=(char*)(_bstr_t)m_pRs->GetCollect("TableUSEID");
	if(bjstr=="0")
	{
		AfxMessageBox("该桌不需要付款");
		return;
	}
	m_ShiShou.GetWindowText(str3);
	if(str3.IsEmpty())
	{
		AfxMessageBox("请输入顾客付款");
		return;
	}
	if(res == TRUE)
		m_Combo.GetWindowText(str);
	else
		m_Combo.GetLBText(m_Combo.GetCurSel(),str);
	m_ZhaoLing.SetWindowText("");
	double zhaoling,rishouru=0;
	m_ShiShou.GetWindowText(ShiShou);
	m_YingShou.GetWindowText(YingShou);
	rishouru=atof(YingShou);
	if(atof(ShiShou)<atof(YingShou))
	{
		AfxMessageBox("想吃霸王餐？");
		return;
	}
	else
	{	
		CTime time;
		time = CTime::GetCurrentTime();
		CString str1 = time.Format("%Y-%m-%d");
		zhaoling=atof(ShiShou)-atof(YingShou);		
		TheValue=(char*)(_bstr_t)zhaoling;		
		m_ZhaoLing.SetWindowText(TheValue);
		UpdateData(false);
		CString sql;
		str2="update TableUSE set TableUSEID=0 where 桌号="+str+" ";
		theApp.m_pCon->Execute((_bstr_t)str2,NULL,adCmdText);
    	TheValue.Format("%0.2f",rishouru);
		sql="update shouru set 日收入=日收入+'"+TheValue+"' where 时间='"+str1+"'";
		theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
		m_YingShou.SetWindowText("");
		m_ShiShou.SetWindowText("");
		m_ZhaoLing.SetWindowText("");
		m_Combo.SetWindowText("");
		m_Combo.DeleteString(m_Combo.GetCurSel());
		m_MingXi.DeleteAllItems();
		sql="delete from paybill where 桌号="+str+"";
		theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
		AfxMessageBox("欢迎再来");					
	}

}

void CJiezhangdlg::OnButtonBye() 
{
	CDialog::OnCancel();
	
}



void CJiezhangdlg::OnChangeEDITshishou() 
{
	double zhaoling;
	CString ShiShou,YingShou;
	m_ShiShou.GetWindowText(ShiShou);
	m_YingShou.GetWindowText(YingShou);
	zhaoling = atof(ShiShou) - atof(YingShou);
	CString str;
	str.Format("%0.2f",zhaoling);
	m_ZhaoLing.SetWindowText(str);

}

void CJiezhangdlg::OnSelchangeCombo1() 
{
	UpdateData();
	CString str,sql,caiming,shuliang,xiaofei,xiaofeitotle,TheValue;
	double totle=0;
	m_Combo.GetLBText(m_Combo.GetCurSel(),str);
	sql="select * from paybill where 桌号="+str+"";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	m_MingXi.DeleteAllItems();
	while(m_pRs->adoEOF==0)
	{
		TheValue=(char*)(_bstr_t)m_pRs->GetCollect("消费");
		totle+=atof(TheValue);
		caiming=(char*)(_bstr_t)m_pRs->GetCollect("菜名");
		shuliang=(char*)(_bstr_t)m_pRs->GetCollect("数量");	
		xiaofei=(char*)(_bstr_t)m_pRs->GetCollect("消费");
		m_MingXi.InsertItem(0,"");
		m_MingXi.SetItemText(0,0,caiming);
		m_MingXi.SetItemText(0,1,shuliang);
		m_MingXi.SetItemText(0,2,xiaofei);
		m_pRs->MoveNext();
	}
	xiaofeitotle=(char*)(_bstr_t)totle;
	m_YingShou.SetWindowText(xiaofeitotle);
	UpdateData(false);
}
void CJiezhangdlg::OnOK()
{
	OnButtonOk2();
}


void CJiezhangdlg::OnEditchangeCombo1() 
{
	m_MingXi.DeleteAllItems();
	m_YingShou.SetWindowText("");
	CString str;
	m_Combo.GetWindowText(str);
	if(str.GetLength()==4)
	{
		UpdateData();
		CString sql,caiming,shuliang,xiaofei,xiaofeitotle,TheValue;
		double totle=0;
		sql="select * from paybill where 桌号="+str+"";
		m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
		while(m_pRs->adoEOF==0)
		{
			TheValue=(char*)(_bstr_t)m_pRs->GetCollect("消费");
			totle+=atof(TheValue);
			caiming=(char*)(_bstr_t)m_pRs->GetCollect("菜名");
			shuliang=(char*)(_bstr_t)m_pRs->GetCollect("数量");	
			xiaofei=(char*)(_bstr_t)m_pRs->GetCollect("消费");
			m_MingXi.InsertItem(0,"");
			m_MingXi.SetItemText(0,0,caiming);
			m_MingXi.SetItemText(0,1,shuliang);
			m_MingXi.SetItemText(0,2,xiaofei);
			m_pRs->MoveNext();
		}
		xiaofeitotle=(char*)(_bstr_t)totle;
		m_YingShou.SetWindowText(xiaofeitotle);
		res = TRUE;
		UpdateData(false);
	}
}


//12 月 11 日

//进货
// Jinhuodlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Jinhuodlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CJinhuodlg dialog

extern CMyApp theApp;
CJinhuodlg::CJinhuodlg(CWnd* pParent /*=NULL*/)
	: CDialog(CJinhuodlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CJinhuodlg)
	//}}AFX_DATA_INIT
}


void CJinhuodlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CJinhuodlg)
	DDX_Control(pDX, IDC_EDIT_totle, m_Totle);
	DDX_Control(pDX, IDC_LIST_foodbill, m_FoodBill);
	DDX_Control(pDX, IDC_LIST_foodlist, m_FoodList);
	DDX_Control(pDX, IDC_EDIT_time, m_Times);
	DDX_Control(pDX, IDC_EDIT_shuliang, m_ShuLiang);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CJinhuodlg, CDialog)
	//{{AFX_MSG_MAP(CJinhuodlg)
	ON_BN_CLICKED(IDC_BUTTON_jinhuo, OnBUTTONjinhuo)
	ON_BN_CLICKED(IDC_BUTTON_tuihuo, OnBUTTONtuihuo)
	ON_BN_CLICKED(IDC_BUTTON_reset, OnBUTTONreset)
	ON_BN_CLICKED(IDC_BUTTON_tuichu, OnBUTTONtuichu)
	ON_BN_CLICKED(IDC_BUTTON_queding, OnBUTTONqueding)

	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CJinhuodlg message handlers

BOOL CJinhuodlg::OnInitDialog()
{
	CDialog::OnInitDialog();
	CTime time;
	time=CTime::GetCurrentTime();
	m_Time=time.Format("%Y-%m-%d");
	m_Times.SetWindowText(m_Time);
	m_FoodList.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_FoodList.InsertColumn(0,"商品名",LVCFMT_LEFT,80,0);
	m_FoodList.InsertColumn(1,"单价(元)",LVCFMT_LEFT,100,1);
	m_pRs=theApp.m_pCon->Execute((_bstr_t)("select * from shangpininfo"),NULL,adCmdText);
	while(!m_pRs->adoEOF)
	{
		CString Value1,Value2;
		Value1=(char*)(_bstr_t)m_pRs->GetCollect("商品名");
		Value2=(char*)(_bstr_t)m_pRs->GetCollect("商品单价");
		m_FoodList.InsertItem(0,"");
		m_FoodList.SetItemText(0,0,Value1);
		m_FoodList.SetItemText(0,1,Value2);		
		m_pRs->MoveNext();
	}
	m_FoodBill.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_FoodBill.InsertColumn(0,"商品名",LVCFMT_LEFT,80,0);
	m_FoodBill.InsertColumn(1,"数量(斤)",LVCFMT_LEFT,100,1);
	m_Totle.SetWindowText("0");
	m_ShuLiang.SetWindowText("1");
	return TRUE;
}
double totle=0;

void CJinhuodlg::OnBUTTONjinhuo() 
{
	UpdateData();
	int n=m_FoodList.GetSelectionMark();
	if(n==-1)
	{
		AfxMessageBox("请选择一样商品");
		return;
	}
	CString name=m_FoodList.GetItemText(n,0);
	CString price=m_FoodList.GetItemText(n,1);
	CString shuliang,Mytotle;
	m_ShuLiang.GetWindowText(shuliang);
	if(shuliang.IsEmpty()||shuliang=="0")
	{
		AfxMessageBox("数量至少为1斤");
		return;
	}
	m_FoodBill.InsertItem(0,"");
	m_FoodBill.SetItemText(0,0,name);
	m_FoodBill.SetItemText(0,1,shuliang);	
	totle+=atof(price)*atof(shuliang);
	Mytotle=(char*)(_bstr_t)totle;
	m_Totle.SetWindowText(Mytotle);
	UpdateData(false);

}

void CJinhuodlg::OnBUTTONtuihuo() 
{
	UpdateData();
	CString Mytotle;
	m_Totle.GetWindowText(Mytotle);
	totle=atof(Mytotle);
	int n=m_FoodBill.GetSelectionMark();
	CString str=m_FoodBill.GetItemText(n,0);
	CString shuliang=m_FoodBill.GetItemText(n,1);
	m_FoodBill.DeleteItem(n);
	CString str1;
	int i=m_FoodList.GetItemCount();
	for(int j=0;j<i;j++)
	{
		str1=m_FoodList.GetItemText(j,0);
		if(str==str1)
		{
			CString price=m_FoodList.GetItemText(j,1);
			totle-=atof(price)*atof(shuliang);
			break;
		}
	}
	Mytotle=(char*)(_bstr_t)totle;
	m_Totle.SetWindowText(Mytotle);
	UpdateData(false);

}



void CJinhuodlg::OnBUTTONreset() 
{
	m_FoodBill.DeleteAllItems();
	m_Totle.SetWindowText("0");
	totle=0;
}

void CJinhuodlg::OnBUTTONtuichu() 
{
	CDialog::OnCancel();
}

void CJinhuodlg::OnBUTTONqueding() 
{
	UpdateData();
	m_Times.GetWindowText(m_Time);
	int i=m_FoodBill.GetItemCount();
	int n=m_FoodList.GetItemCount();
	CString sql,name,shuliang,Mtotle,str,price;
	for(int j=0;j<i;j++)
	{
		name=m_FoodBill.GetItemText(j,0);
		shuliang=m_FoodBill.GetItemText(j,1);	
		for(int k=0;k<n;k++)
		{
			str=m_FoodList.GetItemText(k,0);
			if(name==str)
			{
				price=m_FoodList.GetItemText(k,1);
				totle=atof(price)*atof(shuliang);
				break;
			}
		}
		Mtotle=(char*)(_bstr_t)price;
		sql="insert into jinhuoinfo(进货时间,商品名,商品数量,商品价格)values('"+m_Time+"','"+name+"',"+shuliang+","+Mtotle+")";
		theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	}
	CDialog::OnOK();
}


//进货选择
// Jinhuoselect.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Jinhuoselect.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CJinhuoselect dialog

extern CMyApp theApp;
CJinhuoselect::CJinhuoselect(CWnd* pParent /*=NULL*/)
	: CDialog(CJinhuoselect::IDD, pParent)
{
	//{{AFX_DATA_INIT(CJinhuoselect)
		// NOTE: the ClassWizard will add member initialization here
	//}}AFX_DATA_INIT
}


void CJinhuoselect::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CJinhuoselect)
	DDX_Control(pDX, IDC_LIST1, m_Jinhuoinfo);
	DDX_Control(pDX, IDC_COMBO_year, m_Yearcombo);
	DDX_Control(pDX, IDC_COMBO_month, m_Monthcombo);
	DDX_Control(pDX, IDC_COMBO_day, m_Daycombo);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CJinhuoselect, CDialog)
	//{{AFX_MSG_MAP(CJinhuoselect)
	ON_BN_CLICKED(IDC_CHAXUN_BUTTON, OnChaxunButton)
	ON_BN_CLICKED(IDC_TUICHU_BUTTON, OnTuichuButton)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CJinhuoselect message handlers

BOOL CJinhuoselect::OnInitDialog() 
{
	CDialog::OnInitDialog();	
	CString sql;
	CString str;
	sql="select distinct Year(进货时间) as 年份 from jinhuoinfo ";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		str=(char*)(_bstr_t)m_pRs->GetCollect("年份");
		m_Yearcombo.AddString(str);	
		m_pRs->MoveNext();
	}
	
	sql="select distinct Month(进货时间) as 月份 from jinhuoinfo ";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		str=(char*)(_bstr_t)m_pRs->GetCollect("月份");
		m_Monthcombo.AddString(str);	
		m_pRs->MoveNext();
	}
	
	sql="select distinct Day(进货时间) as 日期 from jinhuoinfo ";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		str=(char*)(_bstr_t)m_pRs->GetCollect("日期");
		m_Daycombo.AddString(str);	
		m_pRs->MoveNext();
	}
	m_Jinhuoinfo.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_Jinhuoinfo.InsertColumn(0,"商品名",LVCFMT_LEFT,60,0);
	m_Jinhuoinfo.InsertColumn(1,"数量(斤)",LVCFMT_LEFT,80,1);
	m_Jinhuoinfo.InsertColumn(2,"价格(元)",LVCFMT_LEFT,80,2);
	return TRUE;  // return TRUE unless you set the focus to a control
	              // EXCEPTION: OCX Property Pages should return FALSE
}

void CJinhuoselect::OnChaxunButton() 
{	
	m_Jinhuoinfo.DeleteAllItems();
	UpdateData();
	CString year,month,day,totle;
	if(m_Yearcombo.GetCurSel()==-1||m_Monthcombo.GetCurSel()==-1||m_Daycombo.GetCurSel()==-1)
	{
		AfxMessageBox("请选择一个时间");
		return;
	}
	m_Yearcombo.GetLBText(m_Yearcombo.GetCurSel(),year);
	m_Monthcombo.GetLBText(m_Monthcombo.GetCurSel(),month);
	m_Daycombo.GetLBText(m_Daycombo.GetCurSel(),day);
	CString sql="select * from jinhuoinfo where Year(进货时间)="+year+" and Month(进货时间)="+month+" and Day(进货时间)="+day+"";
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	while(m_pRs->adoEOF==0)
	{
		CString name,sum,price;
		name=(char*)(_bstr_t)m_pRs->GetCollect("商品名");
		sum=(char*)(_bstr_t)m_pRs->GetCollect("商品数量");
		price=(char*)(_bstr_t)m_pRs->GetCollect("商品价格");
		m_Jinhuoinfo.InsertItem(0,"");
		m_Jinhuoinfo.SetItemText(0,0,name);
		m_Jinhuoinfo.SetItemText(0,1,sum);
		m_Jinhuoinfo.SetItemText(0,2,price);
		m_pRs->MoveNext();
	}
}

void CJinhuoselect::OnTuichuButton() 
{
	CDialog::OnCancel();	
}

void CJinhuoselect::OnOK()
{
	OnChaxunButton();
}
//开台
// Kaitaidlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Kaitaidlg.h"
#include "Diancaidlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CKaitaidlg dialog
extern CMyApp theApp;


CKaitaidlg::CKaitaidlg(CWnd* pParent /*=NULL*/)
	: CDialog(CKaitaidlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CKaitaidlg)
	
	//}}AFX_DATA_INIT
}


void CKaitaidlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CKaitaidlg)
	DDX_Control(pDX, IDC_LIST1, m_Zhuolist);
	DDX_Text(pDX, IDC_EDIT1, m_ZhuoHao);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CKaitaidlg, CDialog)
	//{{AFX_MSG_MAP(CKaitaidlg)
	ON_BN_CLICKED(IDC_BUTTON_OK, OnButtonOk)
	ON_BN_CLICKED(IDC_BUTTON_return, OnBUTTONreturn)
	ON_NOTIFY(NM_DBLCLK, IDC_LIST1, OnDblclkList1)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CKaitaidlg message handlers





void CKaitaidlg::OnButtonOk() 
{
	UpdateData();

	if(m_ZhuoHao.IsEmpty())
		AfxMessageBox("桌号不能为空");
	else
	{	
		CString Str="select * from TableUSE where TableUSEID=1";
		m_pRs=theApp.m_pCon->Execute((_bstr_t)Str,NULL,adCmdText);
		while(!m_pRs->adoEOF)
		{
			Value=(char*)(_bstr_t)m_pRs->GetCollect("桌号");
			if(m_ZhuoHao==Value)
			{
				AfxMessageBox("有人了");
				m_ZhuoHao="";
				UpdateData(false);
				return;
			}
			m_pRs->MoveNext();
		}
		m_pRs=NULL;
		CString Str1="select * from TableUSE where 桌号="+m_ZhuoHao+"";
		m_pRs=theApp.m_pCon->Execute((_bstr_t)Str1,NULL,adCmdText);
		if(m_pRs->adoEOF)
		{
			
			AfxMessageBox("没有这种餐台");
			m_ZhuoHao="";
			UpdateData(false);
			return;
			
		}
		m_pRs=NULL;
		
		CDiancaidlg dlg;
		dlg.m_ZhuoHao = m_ZhuoHao;
		dlg.DoModal();
		CDialog::OnOK();	
	}

}

void CKaitaidlg::OnBUTTONreturn() 
{
	CDialog::OnCancel();	
}
void CKaitaidlg::OnOK()
{
	OnButtonOk();
}

BOOL CKaitaidlg::OnInitDialog() 
{
	CDialog::OnInitDialog();
	SetIcon(LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_ICON_kaitai)),TRUE);
	m_Zhuolist.SetExtendedStyle(LVS_EX_FLATSB|LVS_EX_FULLROWSELECT|LVS_EX_HEADERDRAGDROP|LVS_EX_ONECLICKACTIVATE|LVS_EX_GRIDLINES);
	m_Zhuolist.InsertColumn(0,"桌号",LVCFMT_LEFT,140,0);
	m_Zhuolist.InsertColumn(1,"状态",LVCFMT_LEFT,140,1);
	CString sql="select * from tableuse"; 
	m_pRs=theApp.m_pCon->Execute((_bstr_t)sql,NULL,adCmdText);
	int i=0;
	while(m_pRs->adoEOF==0)
	{
		CString str=(char*)(_bstr_t)m_pRs->GetCollect("桌号");
		int tableuseid=atoi((char*)(_bstr_t)m_pRs->GetCollect("tableuseid"));
		m_Zhuolist.InsertItem(i,"");
		m_Zhuolist.SetItemText(i,0,str);
		if(tableuseid==0)
			m_Zhuolist.SetItemText(i,1,"空闲");
		if(tableuseid==1)
			m_Zhuolist.SetItemText(i,1,"有人");
		i++;
		m_pRs->MoveNext();
	}



	return TRUE;  // return TRUE unless you set the focus to a control
	              // EXCEPTION: OCX Property Pages should return FALSE
}

void CKaitaidlg::OnDblclkList1(NMHDR* pNMHDR, LRESULT* pResult) 
{
	CString str;
	str=m_Zhuolist.GetItemText(m_Zhuolist.GetSelectionMark(),0);
	m_ZhuoHao=str;
	UpdateData(false);
	*pResult = 0;
}


//登录
// Logindlg.cpp : implementation file
//

#include "stdafx.h"
#include "餐饮管理.h"
#include "Logindlg.h"
#include "餐饮管理Dlg.h"

#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif
extern CMyApp theApp;

/////////////////////////////////////////////////////////////////////////////
// CLogindlg dialog


CLogindlg::CLogindlg(CWnd* pParent /*=NULL*/)
	: CDialog(CLogindlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CLogindlg)
	m_Uname = _T("");
	m_Upasswd = _T("");
	//}}AFX_DATA_INIT
	i = 0;
}


void CLogindlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CLogindlg)
	DDX_Text(pDX, IDC_EDIT1, m_Uname);
	DDX_Text(pDX, IDC_EDIT2, m_Upasswd);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CLogindlg, CDialog)
	//{{AFX_MSG_MAP(CLogindlg)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CLogindlg message handlers

void CLogindlg::OnOK() 
{
	UpdateData();
	if(!m_Uname.IsEmpty()||!m_Upasswd.IsEmpty())
	{
		CString sql="SELECT * FROM Login WHERE Uname='"+m_Uname+"' and Upasswd='"+m_Upasswd+"'";
		try
		{			
			m_pRs.CreateInstance("ADODB.Recordset");
			m_pRs->Open((_variant_t)sql,theApp.m_pCon.GetInterfacePtr(),adOpenDynamic,adLockOptimistic,adCmdText);
			if(m_pRs->adoEOF)
			{
				AfxMessageBox("用户名或密码错误!");	
				m_Uname="";
				m_Upasswd="";
				i++;
				UpdateData(false);
				if(i==3)
				{
					OnCancel();		
				}
				
			}
			else
			{
				theApp.name=m_Uname;
				theApp.pwd=m_Upasswd;
				CDialog::OnOK();
				return;
			}
		}
		catch(_com_error e)
		{
			CString temp;
			temp.Format("连接数据库错误信息:%s",e.ErrorMessage());
			AfxMessageBox(temp);
			return;				
		}
	}
	else
	{
		AfxMessageBox("用户名密码不能为空");
	}
}


					



	


BOOL CLogindlg::OnInitDialog() 
{
	CDialog::OnInitDialog();
	SetIcon(LoadIcon(AfxGetInstanceHandle(),MAKEINTRESOURCE(IDI_ICON_login)),TRUE);	
	return TRUE;  // return TRUE unless you set the focus to a control
	              // EXCEPTION: OCX Property Pages should return FALSE
}

void CLogindlg::OnCancel() 
{
	// TODO: Add extra cleanup here
	
	CDialog::OnCancel();
}

